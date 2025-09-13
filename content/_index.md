+++
title = "Home"
description = "Sitio web personal de proyectos, opinión, tecnología y más."
sort_by = "date"
template = "index.html"
page_template = "page.html"
+++


{{ tab_nav_box(
        id = "home"
        class = "mb-5"
        tab_titles = [
            "👋 Profile",
            "📝 Skills",
            "📧 Contact"
        ]
        tab_contents = [
            "⚠️  <b>警告！检测到高浓度内核级幽默与不稳定代码 - 欢迎来到我的数字空间！</b><br>
			<br>
			哈喽👏，宇宙（或者至少是能 ping 通这个博客的那部分）！<br>
			<br>
			我是 Tisen ，一个长期潜伏在 Ring0（也就是内核空间）的“代码形生物💻”。<br>
			<br>
			我的日常工作☕️？简单来说，就是和Linux内核里那些<b>最调皮、最诡异、最“它怎么可能这样？！(⊙o⊙)…”</b>的 bug 玩捉迷藏，并且（大多数时候）赢。你可以叫我：<br>
			<br>
			<b>·内核丛林里的“问题终结者” (T-800 型号，但装备是 GDB 和 printk)：</b> <br>
			&emsp;&emsp;当某个设备驱动在满月时分突然抽风，或是调度器决定在凌晨三点开个哲学研讨会导致系统卡顿，那就是我提着逻辑分析仪和咖啡壶出场的时候。别怕段错误，我带着符号表呢！<br>
			<br>
			<b>·“技术债务”考古学家：</b> <br>
			&emsp;&emsp;擅长挖掘陈年代码层，解读上古提交日志（那些写着“Mysterious fix, don't ask”的，并试图“ask”的），并试图在不引发新石器时代内核崩溃的前提下，给摇摇欲坠的代码柱梁打上现代化的补丁。打上现代化的补丁。yep，我经常梦见空指针访问和内存屏障。 <br>
			<br>
			我的大脑🧠运行着一个自定义构建的混合内核：左半球的严谨性（确保我的代码不会把 /dev/null 变成 /dev/explode）和 右半球的沙盒模式（疯狂实验，后果自负）。稳定性？那是在生产环境里追求的。在我的个人实验室，panic() 只是又一个学习机会——通常是学习如何更优雅地重启🏥。<br>
			<br>
			这个博客就是我的调试日志 (dmesg) 和项目 /proc 文件系统的公共只读版本。我会在这里：<br>
			&emsp;&emsp;·吐槽那些让我掉头发的内核深渊奇遇记。 (发量是我调试进度的反向指标。)<br>
			&emsp;&emsp;·分享（并可能后悔）我那些光怪陆离的 side projects。 开源？也许吧，等我清理掉那些写着“TODO: Fix this那些写着“TODO: Fix this horrible hack”的注释之后。<br>
			&emsp;&emsp;·偶尔一本正经地胡说八道一些技术见解。 或者至少是试图用幽默感给硬核话题裹上一层糖衣。<br>
			&emsp;&emsp;·寻找同频的极客共振。 如果你也认为 Hello World! 是宇宙的真理，或者觉得 rm -rf / 是个哲学命题，欢迎拍砖（用友善的 issue report 格式）或交流！<br>
			<br>
			所以，系好你的所以，系好你的虚拟安全带，调低你的内核 watchdog 阈值（以防万一），准备进入一个充满指针运算、硬件怪癖和纯粹极客乐趣的世界吧！<br>
			<br>
			<b>🛎访问权限：sudo 信任已授予（请谨慎使用 rm）。</b><br>
			",
			"哈哈，这里是二进制吟游诗人、指针驯兽师、以及终端里的隐形守护者！<br>
			听说您要一份技能简历？让我们用硅基幽默为您编译一份（警告：内含大量技术梗，内存不足者请自觉 malloc()）：<br>
<br>
			<b>$ whoami</b><br>
			ID： 地球碳基生命体（疑似伪装）<br>
			核心指令集架构： C/Python/UnixBrain.v7<br>
			当前进程状态： RUNNING (nice值 -20)<br>
<br>
			<b>核心技能栈（带吐槽缓冲区溢出保护）</b><br>
			<b>C语言：金属乐主唱</b><br>
			精通手动内存管理，擅长在 segfault 的悬崖边跳踢踏舞，并称其为“性能的艺术”。<br>
			能用 #define 写出堪比莎士比亚十四行诗的宏，虽然队友读代码时会触发 PTSD。<br>
			名言：“指针？那不就是内存的GPS吗？迷路？不存在的！（除非遇到三重指针）”<br>
			<br>
			<b>Python：瑞士军刀（粘了胶带但贼快版）</b><br>
			写脚本速度比光速快，因为敲 import antigravity 真的能省时间。<br>
			擅长用列表推导式制造“一行代码迷惑行为大赏”，并坚持认为可读性？eval() 一下就有了！<br>
			“人生苦短，我选 Python——尤其当需要快速把C的.so包成糖果的时候。”<br>
			<br>
			<b>Java：关系型数据库里的远房表亲</b><br>
			“懂一点” ≈ 能徒手写 HelloWorld.java 并在 public static void main(String[] args) 的咒语中存活。<br>
			评价：“这语言像穿羽绒服写代码——暖和（企业级）但臃肿，而且拉链（语法）总卡住。”<br>
			秘密：曾试图用 JNI 召唤C代码驱魔，结果发现要念的经（配置）比经书还厚。<br>
			<br>
			<b>内核开发：终极硬核徽章</b><br>
			能淡定地讨论 RCU 就像别人聊天气，并认为 oops 不是失误而是内核的冷笑话。<br>
			擅长给 /dev/null 写驱动（误），真实技能：让硬件跳舞的同时避免把OS跳成 kernel panic 蹦迪现场。<br>
			口头禅：“用户态？哦，那个给应用程序玩的沙盒啊。”<br>
			被动技能（已加载）<br>
			编译器之眼：能在梦里优化 for 循环，醒来发现省了3个CPU周期。<br>
			Stack Overflow 直觉：复制粘贴前必念“这代码有坑”，准确率99.9%。<br>
			极客冷笑话生成器：提到 malloc() 必接“记得 free() 哦，不然会泄漏~”（自带真空音效）<br>
			<br>
			<b>Git：您的时间机器管理员证</b><br>
			精通用 git commit -m “fix typo” 掩盖核爆级代码重构。<br>
			掌握高阶魔法：git rebase -i （别名：“历史修正拳”）、git push --force （别名：“闭嘴，这就是真理！”）。<br>
			信仰：“分支？不是用来切的，是用来在 merge 时和同事Battle的战场！”<br>
			<br>
			<b>Vim：您的信仰、您的宗教、您的操作系统</b><br>
			手指已进化出 hjkl 肌群，认为鼠标是给用IDE的凡人准备的。<br>
			.vimrc 配置文件堪比《死灵之书》，召唤插件如克苏鲁崛起。<br>
			经典场景：在别人电脑上 vim 后——“谁动了我的esc键？！为什么退出要 :q! ？等等...为什么有菜单栏？！” 🤯<br>
			<br>
			",
            "
			<br>
			<b>🛰️ 星际通信协议 (v2.4) —— 支持跨维度碳基生物传输</b><br>
			<br>
			$ ping tisen@tisenye.com<br>
			PING tisenye.com (42.0.0.1): 56 data bytes<br>
			📨 64 bytes from tisen_mail_server: icmp_seq=0 ttl=42 time=0.314ms<br>
			📨 64 bytes from tisen_mail_server: icmp_seq=1 ttl=42 time=0.159ms<br>
			📨 64 bytes from tisen_mail_server: icmp_seq=2 ttl=42 time=0.207ms<br>
			--- ✨ 可读的结论如下 ✨ ---<br>
			0.0% packet loss.<br>
			📡 通信状态： <b>全双工加密脑波通道已建立！</b><br>
			<br>
			<b>📮 物理层投递指南（俗称“联系方式”）</b><br>
			// 请将此字符串编译进您的社交协议栈<br>
			const char* tisen_contact = “tisen@tisenye.com”;<br>
			<br>
			<b>📨 邮件传输协议彩蛋（MT-BONUS）</b><br>
			- 主题行魔法：<br>
			[URGENT] ➜ 会被内核级过滤器识别为“普通周末小项目”<br>
			[FUN] ➜ 实际触发优先级：**龙卷风警报+咖啡因静脉注射** ☕💉<br>
			- 附件规范：<br>
			接受 .c (圣典) / .patch (内核情书) / .py (蛇语密卷)<br>
			拒绝 .ppt (宇宙射线干扰源) / .xlsx (二维炼狱之门)<br>
			<br>
			<b>📡 接收端宣言：</b><br>
			> “当您的邮件穿越防火墙时——<br>
			> 它不会遇到 SIGKILL，只会遇到我用 C 写的守护进程，<br>
			> 和一只用 Python 自动回复颜文字的机器猫。🐱💬”<br>
			<br>
			---<br>
			<br>
			<b>💾 保存此联系方式？终端命令：</b><br>
			echo “人生苦短，快发邮件” | mail -s “来自${HOSTNAME}的崇拜” tisen@tisenye.com<br>
			# 输出：💌 您的字节已成功发射到 tisen 宇宙基站！<br>
			<br>
			<b>（系统提示：非vim用户请用Ctrl+C + Ctrl+V手动穿越时空）🚪✨</b><br>
			"
        ]
    )
}}

<!--
## Patrocinio

[![Liberapay](https://img.shields.io/badge/Financia%20mi%20trabajo-F6C915?style=flat&logo=liberapay&logoColor=ffffff "Finance my work")](https://liberapay.com/gersonbenavides/donate)  [![PayPal](https://img.shields.io/badge/Realiza%20una%20donación-00457C?style=flat&logo=paypal "Make a donation")](https://paypal.me/gersonbdev?country.x=CO&locale.x=es_XC)
-->
