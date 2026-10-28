Mutation4 导演诊断定位版 0.2.1

本版用于定位0.2.0显示game_function_layout_mismatch的具体原因。
当前信息只能确认：三个主要函数地址查询成功，但后续布局检查没有全部通过。
不据此认定Left4DHooks 1.155不兼容，也不要求先升级前置。
本版保留原有字段和调用检查，不会绕过失败项继续读取导演内部字段。

更新

将包内left4dead2与服务器同名目录合并，更新以下两项：
addons/sourcemod/plugins/my_mutation4_diag.smx
addons/sourcemod/gamedata/my_mutation4_diag.txt
gamedata与0.2.0一致，配套提供便于核对；已有cfg配置可继续保留。
源码位于addons/sourcemod/scripting/my_mutation4_diag.sp。

在服务器控制台执行：
sm plugins reload my_mutation4_diag
sm_m4diag_probe

请回贴sm_m4diag_probe的完整输出。暂时不用补传游戏模块或连接服务器。
本命令需要通用管理员权限（b），也可由服务器控制台直接执行。
仅打印版本、校验原因、相关函数地址、少量代码字节和接口可用状态，不包含玩家身份或账号密码。
命令会重新尝试此前缓存的校验失败，但不会修改函数、跳转或移除其他插件的拦截。

新增信息

detail=...：第一项不匹配的具体位置及实际值、预期值。
linux_update_header_mismatch：阶段更新函数头不同。
linux_maximum_header_mismatch / linux_maximum_tail_mismatch：最大intensity getter不同。
linux_lock_header_mismatch：有效锁定判断函数头不同。
linux_maximum_iterator_missing：额外的intensity遍历定位项不可用。
linux_maximum_call_target_mismatch：getter中的调用目标与定位的遍历函数不一致。
linux_*_operand_mismatch / linux_*_opcode_mismatch：具体字段或对应指令不同。

输出中的state=0 timer=0 flow=0可能表示在提取字段位置之前已经停止检查，不是游戏实际的推进值。
在内部读取不可用时，基础阶段、特感和网络intensity采集仍按原逻辑工作，内部数值使用null。

验证范围

已使用SourceMod 1.11.0.6968编译，零错误、零警告。
已按提供的Linux样本核对从源码提取的全部字段检查；原样本通过，8种故障示例能得到对应的具体失败原因。
这些示例不是对当前服务器根因的判定。实际服务器的问题仍待本次probe输出确认。
本版尚未在游戏服务器中执行，不能将离线检查视为运行验证。

源码及许可沿用GPL-3.0-or-later；Left4DHooks接口归Silvers及贡献者所有。
