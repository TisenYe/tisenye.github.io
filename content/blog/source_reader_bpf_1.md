+++
date = "2025-10-20" 
title = "BPF Source reader [1]" 
description = "内核BPF源码走读" 
slug = "bpf_source_reader_1" 
draft = false

[taxonomies] 
  categories = ["source reader", "kernel"] 
  tags = ["BPF"]

[extra] 
  page_identifier = "blog-2025_10_20_1"
+++

> 分析源码版本为 Linux-6.16

## bpf_attr

bpf_attr 是一个 union 其中包含了很多匿名结构体和命名结构体以一种紧凑且模块化的方式表达多个不同的参数集合。在 bpf 中会通过 cmd 来区分如何解析 bpf_attr。

```c
// include/uapi/linux/bpf.h

union bpf_attr {
	// 通过 BPF_MAP_CREATE 来解析
	struct { /* anonymous struct used by BPF_MAP_CREATE command */
		__u32	map_type;	/* one of enum bpf_map_type */
		__u32	key_size;	/* size of key in bytes */
		...
	};
	// 通过 BPF_MAP_*_ELEM 来解析
	struct { /* anonymous struct used by BPF_MAP_*_ELEM commands */
		__u32		map_fd;
		__aligned_u64	key;
		union {
			__aligned_u64 value;
			__aligned_u64 next_key;
		};
		__u64		flags;
	};
	// 通过 BPF_MAP_*_BATCH 来解析
	struct { /* struct used by BPF_MAP_*_BATCH commands */
		__aligned_u64	in_batch;	/* start batch,
						 * NULL to start from beginning
						 */
		...
	} batch;
```

## __sys_bpf

通过 bpf 系统调用  执行 `__sys_bpf` 内核内部的通用实现函数。

```c
// kernel/bpf/syscall.c

// 用户态通过系统调用 bpf 触发 __sys_bpf
SYSCALL_DEFINE3(bpf, int, cmd, union bpf_attr __user *, uattr, unsigned int, size)
{
	return __sys_bpf(cmd, USER_BPFPTR(uattr), size);
}

// 内核态调用 bpf_sys_bpf 来触发 __sys_bpf
BPF_CALL_3(bpf_sys_bpf, int, cmd, union bpf_attr *, attr, u32, attr_size)
{
	switch (cmd) {
	case BPF_MAP_CREATE:
	case BPF_MAP_DELETE_ELEM:
	case BPF_MAP_UPDATE_ELEM:
	case BPF_MAP_FREEZE:
	case BPF_MAP_GET_FD_BY_ID:
	case BPF_PROG_LOAD:
	case BPF_BTF_LOAD:
	case BPF_LINK_CREATE:
	case BPF_RAW_TRACEPOINT_OPEN:
		break;
	default:
		return -EINVAL;
	}
	return __sys_bpf(cmd, KERNEL_BPFPTR(attr), attr_size);
}
```

`__sys_bpf()` 是 Linux 内核中 **BPF 系统调用实现的核心函数**。

它是 BPF 子系统的“系统调用调度器”指挥 bpf 指令安全的前往目的地，它做了三件事：

1. **安全检查与参数复制（用户态 or 内核态）**
2. **调用安全框架（LSM hook）**
3. **根据命令号分派到具体实现函数**

```c
// kernel/bpf/syscall.c

static int __sys_bpf(enum bpf_cmd cmd, bpfptr_t uattr, unsigned int size)
{
	union bpf_attr attr;
	int err;

	/* bpf_check_uarg_tail_zero 是一个安全检查函数。
   	     * 参数 uattr 是一个 bpfptr_t，可能来自用户空间，也可能来自内核空间。
   	     * 它的主要作用是确保用户空间传递过来的结构体参数，
   	     * 如果比内核已知的结构体更大，多出来的部分必须全部为零。
   	     * 这样可以防止新版本用户空间依赖内核尚未支持的扩展字段，保证内核的兼容性和安全性。
   	     */
	err = bpf_check_uarg_tail_zero(uattr, sizeof(attr), size);
	if (err)
		return err;
	size = min_t(u32, size, sizeof(attr));

	/* copy attributes from user space, may be less than sizeof(bpf_attr) */
	memset(&attr, 0, sizeof(attr));
	if (copy_from_bpfptr(&attr, uattr, size) != 0)
		return -EFAULT;

	/* security_bpf实现了对 LSM（Linux Security Module）的注册。
	 * 通过 call_int_hook(bpf, cmd, attr, size, kernel); 宏
	 * 分发给所有注册的安全模块（SELinux、AppArmor、Landlock 等）去审查。
	 */
	err = security_bpf(cmd, &attr, size, uattr.is_kernel);
	if (err < 0)
		return err;

	/* bpf 具体实现的入口，通过 cmd 来区分作用
	 */
	switch (cmd) {
          /* Map 是 BPF 的“共享内存”，用于用户态和 BPF 程序之间传递数据。
           * 下面的都是 Map 的增删改查等基础操作接口。
           */
	case BPF_MAP_CREATE:
		err = map_create(&attr, uattr.is_kernel);
		break;
	case BPF_MAP_LOOKUP_ELEM:
		err = map_lookup_elem(&attr);
		break;
	case BPF_MAP_UPDATE_ELEM:
		err = map_update_elem(&attr, uattr);
		break;
	case BPF_MAP_DELETE_ELEM:
		err = map_delete_elem(&attr, uattr);
		break;
	case BPF_MAP_GET_NEXT_KEY:
		err = map_get_next_key(&attr);
		break;
	case BPF_MAP_FREEZE:
		err = map_freeze(&attr);
		break;

	/* 将 BPF 程序加载进内核，经过 verifier 验证后生成一个可执行的内核对象。返回一个 FD。
	 */
	case BPF_PROG_LOAD:
		err = bpf_prog_load(&attr, uattr, size);
		break;
         /* 把 BPF 对象（map/prog/link）固定（pin）到 /sys/fs/bpf，实现跨进程持久化。
          */
	case BPF_OBJ_PIN:
		err = bpf_obj_pin(&attr);
		break;
         /* 从 bpffs 路径重新打开一个 pinned 对象。
          */
	case BPF_OBJ_GET:
		err = bpf_obj_get(&attr);
		break;
         /* 把程序 attach/detach 到某个 hook 点（如 cgroup、kprobe、tracepoint、XDP、tc 等）。
          */
	case BPF_PROG_ATTACH:
		err = bpf_prog_attach(&attr);
		break;
	case BPF_PROG_DETACH:
		err = bpf_prog_detach(&attr);
		break;
          /* 查询某个 hook 上挂了哪些程序。
           */
	case BPF_PROG_QUERY:
		err = bpf_prog_query(&attr, uattr.user);
		break;
	case BPF_PROG_TEST_RUN:
		err = bpf_prog_test_run(&attr, uattr.user);
		break;
	case BPF_PROG_GET_NEXT_ID:
		err = bpf_obj_get_next_id(&attr, uattr.user,
					  &prog_idr, &prog_idr_lock);
		break;
	case BPF_MAP_GET_NEXT_ID:
		err = bpf_obj_get_next_id(&attr, uattr.user,
					  &map_idr, &map_idr_lock);
		break;
	case BPF_BTF_GET_NEXT_ID:
		err = bpf_obj_get_next_id(&attr, uattr.user,
					  &btf_idr, &btf_idr_lock);
		break;
	case BPF_PROG_GET_FD_BY_ID:
		err = bpf_prog_get_fd_by_id(&attr);
		break;
	case BPF_MAP_GET_FD_BY_ID:
		err = bpf_map_get_fd_by_id(&attr);
		break;
	case BPF_OBJ_GET_INFO_BY_FD:
		err = bpf_obj_get_info_by_fd(&attr, uattr.user);
		break;
	case BPF_RAW_TRACEPOINT_OPEN:
		err = bpf_raw_tracepoint_open(&attr);
		break;
          /* BTF 是 BPF 的类型系统描述信息，
           * 用于支持高级调试、符号、CO-RE（Compile Once, Run Everywhere）。
           * BPF_BTF_LOAD 把 BTF 类型信息加载进内核。
           */
	case BPF_BTF_LOAD:
		err = bpf_btf_load(&attr, uattr, size);
		break;
	case BPF_BTF_GET_FD_BY_ID:
		err = bpf_btf_get_fd_by_id(&attr);
		break;
	case BPF_TASK_FD_QUERY:
		err = bpf_task_fd_query(&attr, uattr.user);
		break;
	case BPF_MAP_LOOKUP_AND_DELETE_ELEM:
		err = map_lookup_and_delete_elem(&attr);
		break;
         /* 批量操作 map，性能优化接口。
          */
	case BPF_MAP_LOOKUP_BATCH:
		err = bpf_map_do_batch(&attr, uattr.user, BPF_MAP_LOOKUP_BATCH);
		break;
	case BPF_MAP_LOOKUP_AND_DELETE_BATCH:
		err = bpf_map_do_batch(&attr, uattr.user,
				       BPF_MAP_LOOKUP_AND_DELETE_BATCH);
		break;
	case BPF_MAP_UPDATE_BATCH:
		err = bpf_map_do_batch(&attr, uattr.user, BPF_MAP_UPDATE_BATCH);
		break;
	case BPF_MAP_DELETE_BATCH:
		err = bpf_map_do_batch(&attr, uattr.user, BPF_MAP_DELETE_BATCH);
		break;
          /* BPF Link 是一种“持久 attach”机制。
           * 它把“程序-挂载点”的绑定变成一个独立对象，可安全更新、替换、销毁。
           * 创建一个 link，把 BPF 程序挂到指定 hook（tracepoint、LSM、cgroup、perf_event 等）。
           */
	case BPF_LINK_CREATE:
		err = link_create(&attr, uattr);
		break;
	case BPF_LINK_UPDATE:
		err = link_update(&attr);
		break;
	case BPF_LINK_GET_FD_BY_ID:
		err = bpf_link_get_fd_by_id(&attr);
		break;
	case BPF_LINK_GET_NEXT_ID:
		err = bpf_obj_get_next_id(&attr, uattr.user,
					  &link_idr, &link_idr_lock);
		break;
         /* 启用全局 BPF 统计数据（运行次数、验证器信息等）。
          */
	case BPF_ENABLE_STATS:
		err = bpf_enable_stats(&attr);
		break;
         /* 创建 BPF iterator（用 BPF 迭代内核数据结构，比如任务、sockets）。
          */
	case BPF_ITER_CREATE:
		err = bpf_iter_create(&attr);
		break;
	case BPF_LINK_DETACH:
		err = link_detach(&attr);
		break;
         /* 将 map 绑定到程序上，作为其持久依赖（防止被意外释放）。
          */
	case BPF_PROG_BIND_MAP:
		err = bpf_prog_bind_map(&attr);
		break;
	case BPF_TOKEN_CREATE:
		err = token_create(&attr);
		break;
	default:
		err = -EINVAL;
		break;
	}

	return err;
}
```



## bpftrace 命令的运行过程——内核BPF逻辑

`bpftrace` 是一个前端，通过编译字节码将BPF指令送入内核，由内核执行具体的BPF逻辑，其过程如下：

1. 它把脚本（`BEGIN {}`, `kprobe:xxx {}`）解析成 **AST（抽象语法树）**。

2. 然后将其编译为 **BPF 字节码（BPF 指令集）**。

    - BPF 是一套独立于架构的指令系统，有自己的寄存器模型和栈。

    - 指令例如：

        ```asm
        r1 = ctx
        call bpf_get_current_comm
        call bpf_trace_printk
        ```

3. `bpftrace` 通过 `libbpf` 或 `bpf()` 系统调用将这些字节码送入内核：

    ```c
    syscall(__NR_bpf, BPF_PROG_LOAD, &attr, sizeof(attr));
    ```

    `attr` 中包含：

    - 程序类型（如 `BPF_PROG_TYPE_KPROBE`）
    - 指令指针和大小
    - license、附加选项等信息

4. 系统调用 `bpf()` 通过 `BPF_PROG_LOAD` 执行 `bpf_prog_load`

    ```c
    case BPF_BTF_LOAD:
    	err = bpf_btf_load(&attr, uattr, size);
    	break;
    ```

5. 加载阶段：`bpf_prog_load()`

    主要流程：

    1. **分配内核内存** 用于存放 BPF 指令代码。
    2. **根据程序类型**（如 KPROBE、TRACEPOINT、XDP）选择一个运行环境。
    3. **调用 verifier（验证器）** 进行安全审查。

    - 验证内容包括：
        - 是否有非法内存访问。
        - 是否可能死循环。
        - 栈空间是否越界。
        - 函数调用是否合法。
        - map 操作是否类型匹配。
    - 验证器会构建一个 **控制流图（CFG）** 并模拟执行每条指令。

    4. **如果通过验证**，就会把字节码交给 JIT 编译器（`bpf_prog_select_runtime`）。

6. JIT 编译阶段：`bpf_int_jit_compile()`

    JIT（Just-In-Time 编译器）将 BPF 字节码翻译成当前 CPU 架构的原生机器码。

    举个例子：

    ```asm
    r0 = 0
    call bpf_get_current_comm
    ```

    在 ARM64 上可能变成：

    ```asm
    mov x0, sp
    bl  <bpf_get_current_comm>
    mov x0, #0
    ret
    ```

    这样一来，执行效率几乎等同于原生内核函数。

    JIT 编译的结果会缓存下来（/sys/kernel/debug/tracing/jit_dump 可查看）。

7. Attach 阶段：建立触发点（Hook）

    加载完 BPF 程序后，bpftrace 再调用：

    ```c
    sys_bpf(BPF_RAW_TRACEPOINT_OPEN, &attr, sizeof(attr));
    ```

    或

    ```c
    sys_bpf(BPF_LINK_CREATE, &attr, sizeof(attr));
    ```

    这些命令会把 BPF 程序挂到指定的 hook：

    - `kprobe:sys_open` → kernel probe，拦截内核函数入口
    - `tracepoint:sched:sched_process_exec` → tracepoint hook
    - `uprobe:/bin/bash:malloc` → user-space probe
    - `BEGIN {}` → tracepoint “fake event”

    内核通过 `bpf_link` 或 `perf_event` 把这个程序注册到对应钩子点。
     当钩子被触发时，BPF 程序就会执行。

8. 执行阶段：内核执行 BPF 程序

    当事件发生（比如 `sys_open()` 被调用）时：

    1. 内核执行 kprobe 框架中的通知逻辑：

        ``` c
        for each attached BPF program:
            run_bpf_prog(prog, ctx)
        ```

    2. `run_bpf_prog()` 进入 `bpf_prog_run_xxx()`，执行 BPF 虚拟机或已 JIT 的机器码。

    3. BPF 程序通过 helper 调用访问内核信息：

        - `bpf_get_current_pid_tgid()`
        - `bpf_trace_printk()`
        - `bpf_map_lookup_elem()`
        - `bpf_ktime_get_ns()` 等。

    4. 程序运行结果可能：

        - 写入 BPF map
        - 通过 perfbuf 或 ringbuf 发消息到用户态
        - 返回过滤判断（比如 XDP drop/pass）

9. 用户态收集结果

    bpftrace 监听这些 buffer：

    - **perf event buffer**：事件型输出（比如 `printf()`）
    - **maps**：周期性查询统计数据（比如 `@count`）
    - **ring buffer**：零拷贝高性能传输。

    bpftrace 会在后台用 epoll 监听这些文件描述符，当有数据可读时输出结果。

10. 结束与卸载

    当用户 Ctrl+C 或程序退出：

    1. bpftrace 调用 `close(fd)`。

    2. 内核引用计数减少。

    3. 当没有引用时：

        - 程序（`bpf_prog`）
        - maps
        - link

        都会通过 RCU 延迟释放。


