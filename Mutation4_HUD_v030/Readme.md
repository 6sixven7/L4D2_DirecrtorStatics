Mutation4 Director HUD 0.3.0

本版在 0.2.2 诊断插件上接入全服 EMS HUD，参考 lyt_ems_hud.sp 与 sorallll 的 l4d2_ems_hud.inc。沿用原插件文件名，可覆盖更新；不要同时加载多个改名副本。

画面内容
- 当前 Tempo、下一阶段名称、LockTempo 锁定状态。
- 当前阶段实际剩余时间；Peak Fade 明确标为不使用转阶段计时。
- 当前阶段推进 flow / 转阶段 flow 阈值 / 剩余 flow；只有 Peak Fade 与 Relax 使用，其他阶段明确标为不使用。
- 下波自然尸潮计时器。已到时、未启动和未知分别显示，计时到零不保证立即刷出尸潮。
- 整体路程百分比：沿用 Coop Markers 算法，对存活且有有效导航数据的生还者取当前 flow 平均值，除以地图总 flow，再向上取整。不是累加所有人的距离，也不是最远生还者百分比。
- 所有生还者的强度，包含 BOT。每页 4 人，超过 4 人默认每 5 游戏秒轮页；不限制总人数。名字适度缩短，保留完整 UTF-8 字符，用 #客户端编号区分同名。死亡者显示死亡，不展示可能过期的强度。

按本次要求，HUD 的阶段读数只列时间、flow 和锁定，不替玩家判断转阶段条件是否满足。时间已到但仍未转阶段可能与强度、Tank、LockTempo 等有关；日志继续保留完整条件和原始读数。flow 转阶段采用严格超过阈值，显示舍入到一位小数，不能仅凭画面刚到 0.0 推断已经超过。

转阶段 flow 与整体路程是两个独立读数。原始 flow 允许停滞、下降或剩余距离增加，不修复地图导航特色。负计时表示原始计时已过期，不强制改成 0。阈值从当前脚本及 cvar 读取，不写死为 3000。极大数值仅缩写显示，不限制真实参数。

兼容与依赖
- 运行需要 L4D2、SourceMod、Left4DHooks。使用 SourceMod 1.11.0.6968 编译；原 0.2.2 在你的 Linux、Left4DHooks 1.168、coop 环境已成功读数。
- 沿用经校验的 Windows x86 / Linux x86 内部读取，不承诺 64 位或任意游戏更新后的内存布局。不匹配时只将相关读数显示未知，基础 HUD 和生还者强度仍可显示。
- HUD 本版尚未在真实游戏内验证；coop、Mutation4、第三方地图的显示效果和其他 HUD 同服效果仍需实测。之前的 coop 读数测试不等于 Mutation4 实测。
- 无需客户端 VPK。Hud 字体、比例与可见效果受客户端设置影响，可调整面板位置和行距。
- 与 Coop Markers 参考一致，发现 trigger_finale 的终章不显示路程百分比，显示“终章不适用”。地图总 flow 无效或无存活且导航有效的生还者时显示未知。
- 读取导演，不修改导演的刷怪或阶段参数。EMS 显示按参考插件设置共享的 m_bChallengeModeActive 显示开关；卸载/关闭时不将这个共享开关强制关闭，以免影响其他 HUD。

共享 HUD 栏位
EMS 总共 15 个栏位，本插件使用 6 个空闲栏位，默认自动选择。只清理仍能确认属于自己的内容，不调用清除所有 HUD 的操作。其他插件覆盖本插件的栏位时，本插件让出并在 15 游戏秒后重试。

lyt_ems_hud.sp 仅作为显示实现参考，并不是本插件必须安装的前置。它的排行榜和击杀列表同时开启时会占用大量栏位，可能没有 6 个空位。此时本插件等待，不强抢对方内容。请关闭对方的部分 HUD 功能，或安排 6 个双方约定的空栏位后在本配置中指定。不同栏位的画面仍可能重叠，可调整本面板位置。若其他脚本在稍后重新占用栏位，依然可能出现被接管后短暂消失的情况；没有通用的跨插件栏位预留协议。

安装及确认
1. 将包内 left4dead2/ 与服务器同名目录合并，先备份将被覆盖的旧配置和插件。保留你需要的原有诊断设置，不同时加载旧版改名副本。
2. 更新的是 my_mutation4_diag.smx、对应源码/EMS include、同名 gamedata 及配置。HUD 不需要 lyt 的 gamedata，也不复制它的排行榜/击杀统计或休眠挂钩。
3. 换图加载；或在已运行的游戏中由管理员重载 my_mutation4_diag。HUD 可在地图运行中加载，默认下一次刷新显示；尚需你的服务器核对实际画面。
4. 服务器控制台 sm_m4hud_status 检查状态。active 为已取得栏位；need_six_free_slots_or_check_config 表示空位不足或栏位配置有误；slot_overwritten_retry_15s 表示被其他 HUD 接管；disabled 表示已关闭。
5. 服务器控制台 sm_m4diag_status 检查内部读取；遇 tempo_crosscheck_pending 是阶段切换同步等待，当次相关读数显示未知，不借用旧阶段数据。

玩家指令与权限
- sm_dhud：普通玩家/游客可在客户端控制台执行，也可聊天输入 !dhud；服务器控制台同样可执行。开关全服整个导演 HUD，与参考插件一样属于共享开关。不是个人隐藏按钮；任何玩家切换都会影响全服。
- sm_m4hud_status：管理员（通用权限 b）或服务器控制台查看 HUD 状态。
- sm_m4diag_status / sm_m4diag_snapshot / sm_m4diag_probe：保留已有管理员或服务器控制台诊断指令。

配置与日志
配置位置：left4dead2/cfg/sourcemod/my_mutation4_diag.cfg。
修改后服务器控制台执行 exec sourcemod/my_mutation4_diag.cfg。全部选项、默认值和单位写在配置注释中。sm_dhud 改变当前运行状态，下次执行配置时恢复文件中的开关设定。
HUD 与日志有独立开关和刷新间隔；关 HUD 后日志继续记录，关自动日志后 HUD 继续刷新。
日志仍写入 left4dead2/addons/sourcemod/logs/my_mutation4_diag/m4diag_YYYYMMDD.jsonl，新增 progress.status / contributors / percent 实际进度，其他字段继续保留。旧版本日志仍会留在同日文件中。

验证记录
已用 SourceMod 1.11.0.6968 编译通过，零错误、零警告；已核对包内源码、SMX、配置、gamedata 和 include 的一致性，以及参考算法、独立开关、无管理员限制的 sm_dhud、UTF-8 名字截断和分页边界。尚无连接中的游戏服务器，不能声称已验证画面排版或同服兼容。
