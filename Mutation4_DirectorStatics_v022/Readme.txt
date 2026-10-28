Mutation4 导演诊断修订版 0.2.2

本版处理0.2.1报告的：
linux_lock_header_mismatch / actual=E9 expected=55。

已确认服务器中ShouldLockTempo入口是E9 rel32跳转；具体安装拦截的插件尚未确定。
0.2.x原先只接受未被拦截的入口，导致内部采集在此处停止。

本版的读取方式

1. 继续核对阶段更新函数、最大intensity函数、字段关系和导演虚表。
2. Linux有效锁定判断支持原始入口，以及首5字节被E9 rel32替换的入口。
3. 核对ShouldLockTempo完整121字节范围内的其余代码；仅忽略ELF装载时会重定位的字符串地址。
4. 每次内部采样前重新检查锁定函数，函数体出现其他差异时仍停止内部采集。
5. 通过游戏原入口调用getter，让现有拦截参与最终返回值计算。
6. 不读取跳转目标的代码，不调用跳转后的中间位置，不移除、覆盖或恢复其他插件的拦截。

版本核对

此前日志中的Left4DHooks版本来自服务器left4dhooks_version变量，不是从本地include或gamedata文件名推断。
本版优先使用Left4DHooks_Version()原生接口，同时保留版本变量读数：
left4dhooks_native=1168表示原生接口报告1.168；1155表示1.155。
version_cvar=...是版本变量。
version_source=native表示顶部Left4DHooks版本显示采用原生接口；接口缺失时才回退到变量。
若两者不一致，先核对服务器实际加载的插件，不仅检查磁盘上的源码和include。
官方1.168源码加载时会主动更新版本变量，因此普通热更新旧值残留不能作为默认解释。
前置SMX和其配套gamedata应相互匹配；当前E9入口拦截的兼容问题仍需独立验证。
本包不替换或升级Left4DHooks本身。

安装或更新

将包内left4dead2目录与服务器同名目录合并，更新my_mutation4_diag.smx及配套gamedata。
本版gamedata与0.2.1一致，配套提供便于核对；已有cfg参数名不变，保留自定义配置即可。
源码位于addons/sourcemod/scripting/my_mutation4_diag.sp。
文件路径相对于服务器left4dead2目录。

服务器控制台执行：
sm plugins reload my_mutation4_diag
sm_m4diag_probe
sm_m4diag_snapshot

本次已报告的E9入口通过其余检查时，预计显示：
internal=linux_x86_lock_jump_validated
live_read=1
lock_entry_body_checked=1 entry_rel32_jump=1
如果变为linux_lock_body_mismatch，请提供detail指出的具体字节及完整probe输出。
如果只是tempo_crosscheck_pending，可在阶段稳定后重试。

诊断命令需要通用管理员权限（b）或服务器控制台权限。
sm_m4diag_probe输出包括版本、校验项、函数头和接口状态，不包含玩家身份和账号密码。
请回贴probe完整输出，尤其是原生版本和版本变量；随后核对生成的快照。
当前不用补传游戏模块或连接服务器。

日志

addons/sourcemod/logs/my_mutation4_diag/m4diag_YYYYMMDD.jsonl
仍约每秒记录一次；路程百分比和HUD继续预留。
新增runtime.lock_entry_body_checked、runtime.lock_entry_rel32_jump和版本来源信息。
lock_entry_body_checked只表示代码校验通过，不表示跳转目标所属插件已经识别，也不等于服务器实测已完成。
内部读取失败时保留基础采集，内部值使用null。

已完成验证

SourceMod 1.11.0.6968编译：零错误、零警告。
原始Linux样本和模拟装载重定位检查通过。
将本次输出的32字节E9函数头放入已核实样本后，兼容检查通过。
7处函数体扰动、未知入口、自跳转、空跳转等10种异常被拒绝。
保留原符号入口调用和虚表交叉核对，未加入内存写入或新的detour。
以上为离线重放和静态检查；本版尚待实际服务器运行确认。

许可沿用GPL-3.0-or-later；Left4DHooks接口归Silvers及贡献者所有。
