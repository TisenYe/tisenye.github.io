+++
date = "2025-02-10" 
title = "Docker Desktop进行 BPF 开发" 
description = "" 
slug = "docker bpf dev" 
draft = false

[taxonomies] 
  categories = ["devlop"]
  tags = ["docker", "BPF"]

[extra]
  page_identifier = "blog-2025-01"
+++

通过Dockerfile.tools 来构建相应镜像，参考:

[使用 Docker Desktop进行 BPF 开发](https://luckymrwang.github.io/2022/05/23/%E4%BD%BF%E7%94%A8-Docker-Desktop%E8%BF%9B%E8%A1%8C-BPF-%E5%BC%80%E5%8F%91/)

原文中的 Dockerfile.tools 有些地方需要修改一下

-   有个 llvm 版本的报错 `undefined reference to getPollyPluginInfo()`
-   需要添加 zip 包
-   通过 sed bcc/__init__.py 的方式由于安装方式为 egg，因此无法直接 sed 需要注释掉，该[bug已被修复](https://github.com/iovisor/gobpf/pull/321/commits/1d78ac6cb237a6003e8f85b5abed1d2c297ca4cd)

其中的**for-desktop-kernel**是我们从docker hub下载的内核相关的包资源。

```bash
FROM docker/for-desktop-kernel:5.10.47-0b705d955f5e283f62583c4e227d64a7924c138f AS ksrc

FROM ubuntu:20.04 AS bpftrace
COPY --from=ksrc /kernel-dev.tar /
RUN tar xf kernel-dev.tar && rm kernel-dev.tar

# 使用阿里云镜像加速 Ubuntu 软件源
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list

# 安装 LLVM 10（确保版本至少为 10.0.1）及 Clang
RUN apt-get update && apt-get install -y wget zip lsb-release software-properties-common && \
    wget https://apt.llvm.org/llvm.sh && chmod +x llvm.sh && \
    ./llvm.sh 10 && \
    apt-get install -y llvm-10 clang-10 libclang-10-dev

ENV PATH "$PATH:/usr/lib/llvm-10/bin"

# 安装 bpftrace
RUN apt-get install -y bpftrace

# 安装 bcc 的构建依赖，并创建 Python 的软链接
WORKDIR /root
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y \
      kmod vim bison build-essential cmake flex git libedit-dev \
      libcap-dev zlib1g-dev libelf-dev libfl-dev \
      python3.8 python3-pip python3.8-dev clang libclang-dev && \
    ln -s $(which python3) /usr/bin/python

# 克隆 bcc 源码并对其 CMake 配置进行 patch，追加 Polly 库到链接列表
RUN git clone https://github.com/iovisor/bcc.git && \
    cd bcc && \
    echo "list(APPEND clang_libs Polly PollyISL)" >> cmake/clang_libs.cmake

# 编译并安装 bcc（包含 C++ 和 Python 部分）
WORKDIR /root/bcc/build
RUN cmake .. && make && make install && \
    cmake -DPYTHON_CMD=python3 .. && \
    cd src/python/ && make && make install && \
    #sed -i "s/self._syscall_prefixes\[0\]/self._syscall_prefixes\[1\]/g" /usr/lib/python3/dist-packages/bcc/__init__.py

# 容器启动时挂载 debugfs 并进入 bash
CMD mount -t debugfs debugfs /sys/kernel/debug && /bin/bash
```

原文中运行 ebpf-for-mac 是为了调试当前mac的应用进程，因此需要安装内核版本一致的相关资源。

```bash
$ docker run -it --rm \
  --name ebpf-for-mac \
  --privileged \
  -v /lib/modules:/lib/modules:ro \
  -v /etc/localtime:/etc/localtime:ro \
  --pid=host \
  ebpf-for-mac
```

如果只是为了使用bpf需要修改一下。

```bash
$ docker run -it --rm \
  --name ebpf-for-mac \
  --privileged \
  -v /etc/localtime:/etc/localtime:ro \
  --pid=host \
  ebpf-for-mac
```

进入docker后需要手动链接 build 目录

`ln -sf /usr/src/linux-headers-5.10.47-linuxkit/ /lib/modules/6.10.14-linuxkit/build`

保存 docker

```bash
# 
docker commit $(CONTAINER ID) ebpf-for-mac:latest


```

```c
static struct mmc_blk_data *mmc_blk_alloc_req(struct mmc_card *card,
					      struct device *parent,
					      sector_t size,
					      bool default_ro,
					      const char *subname,
					      int area_type)
{
	struct mmc_blk_data *md;
	int devidx, ret;

	devidx = ida_simple_get(&mmc_blk_ida, 0, max_devices, GFP_KERNEL);
	if (devidx < 0) {
		/*
		 * We get -ENOSPC because there are no more any available
		 * devidx. The reason may be that, either userspace haven't yet
		 * unmounted the partitions, which postpones mmc_blk_release()
		 * from being called, or the device has more partitions than
		 * what we support.
		 */
		if (devidx == -ENOSPC)
			dev_err(mmc_dev(card->host),
				"no more device IDs available\n");

		return ERR_PTR(devidx);
	}

	md = kzalloc(sizeof(struct mmc_blk_data), GFP_KERNEL);
	if (!md) {
		ret = -ENOMEM;
		goto out;
	}
```

