# 原生界面与完整操作入口

本界面由 Godot Control、Container、ScrollContainer、Button、TextureRect 与原生场景构成，没有 HTML / WebView。背景画面不会作为可建平台使用；仙府由 EstateView 的真实格坐标、独立建筑与选择/搬迁信号构成。

## 固定五项导航

- 仙府：点击建筑管理、空地建设/扩地、拖动布局后确认、取消、驻府、升级、队列、收取、离线回报。
- 万仙：动态读取全部图鉴；详情、角色实际修为投入与免费重置、分支、灵宝、五部位灵装、真实面板战力、法相试炼、编队与三份预设。
- 历练：12节点主线敌情、两条五节点秘境、三类无收益演武；实际手动目标/普攻/神通/护持、玄都来源/接受者选择、三霄普通/合阵、法门、自动与1/2/3倍速度、实际战报与原子结算。
- 召请：单抽/十连、三档概率与90/10保底、重复道缘、记录、心愿、定向兑换。概率与成本读取 GameEconomy.gacha_config，不维护第二份数值。
- 事务：签到、日常/成长任务、固定本地奖励信、成长活动、背包与道具、派遣、普通资源商店、引雷工程、设置/导入导出/有效备份恢复，以及明确标注的本机联机流程模拟。

## 新增系统的完整 UI

灵装展示主/副词条、2/4件套、锁定/收藏、按部位选择、排序、装备与卸下、强化前费用和每4级随机规则、材料消耗列表、溢出返还/损失、分解精华预览。数值由 GameEquipment 读取。战力不是胜负阈值。

当前培养演武包含真实装备属性快照；标准化演武不借用正式装备。未完成法相试炼在历练页始终提供锁定快照继续与主动结束入口，避免调整培养后陷入进度死锁。

模拟账号有普通规则/试用规则分区，test_a/test_b 与两个虚拟设备。每个设备保留独立本地缓存，普通账号资产不参与测试。模拟云档比较版本、导入前校验并备份；冲突由玩家明确选择。聊天、公会、榜单只包含本机账号真实执行的测试内容。支付仅生成本机测试订单及回调，绝不真实扣款；领奖调用 GameStore.redeem_mock_receipt，重新核验服务内回执且去重。

## 视觉与资源

青黛背景、玉色信息、暖金主动作；标题使用 Noto Serif SC，正文使用 Noto Sans SC。正文、图标和界面均独立渲染，图片保持比例。

assets/ui 中的宫阙、图鉴、交锋、召请符、事务箱、五类灵装、签到/邮件/云档/公会/榜单、六资源、设置与九宫格边框均为本项目原创 SVG；不是商业游戏资源提取。可直接编辑 SVG 与 scripts/game_theme.gd。

实际检查了两类官方展示参考，仅借鉴层级/物件卡/确认交互：
- 自在西游官方站：https://zzxy.yokaverse.com/home
- 自在西游官方入驻 TapTap 展示：https://www.taptap.cn/app/240118
- 上述页面截图：https://img-tc.tapimg.com/market/images/5d592022ae9b8aad40bfc6139f6a5497.jpg/_tap_screenshot.jpg
- 疯狂水世界官方发行页面：https://game.xiaomi.com/game/62416703
- 上述页面建设图：https://t1.g.mi.com/thumbnail/jpeg/w600h60/AppStore/0e7fa7896591e41daae926004548e45fe6c413625

参考图仅供本地设计审查，docs/ui-reference/.gdignore 阻止作为运行素材导入；发布包应排除该目录。游戏运行不依赖外站或参考图片。

## 验证入口

- `godot --headless --path . --script tests/test_ui.gd -- --ui-test`
- 真实原生窗口截图：`godot --path . --script tests/capture_ui.gd -- --ui-test`
- GUI 从 Terminal 启动方便入口：`sh tests/preview.sh tests/capture_ui.gd`

两个测试脚本均自动标记独立测试存档目录，即使忘记 --ui-test 参数也不会使用正常存档。每次测试目录以时钟与计时器唯一标识，避免沙箱重复 PID 导致读回旧测试档。

截图来自真正 Godot 运行时视口，非网页、设计图或预渲染效果：docs/ui-estate-polished.png、ui-roster-polished.png、ui-character-polished.png、ui-summon-polished.png、ui-affairs-polished.png、ui-battle-polished.png、ui-mock-login-polished.png。原生捕获分辨率540×960。需要重新运行截图入口才能反映后续资源修改。

基础测试遍历360/540/800逻辑宽度、全部图鉴详情、五页与主要弹层，执行召请、建设、首场真实手动+自动战斗、结算、装备、六展示槽、秘境、模拟账号/云档/聊天/公会/榜单/支付回调/去重，最后检查普通资产未被模拟分区影响。不得将这套UI冒烟等同于完整产品全部规则测试；规则各自的测试和正式视觉审查另行记录。

运行环境没有可用声卡时 Godot 自动落到 Dummy 驱动；程序音频已实现，当前云端不能据此宣称真机听感完成验收。

## 事件驱动战斗表现与恢复

`scripts/battle_effects.gd` 只读战斗已提交事件。杨戬先开天眼并点亮标记，再出枪；赵公明的五珠绕身、发射、返还；三霄依实际 `responderUids` 依次亮符并连阵，不生成额外行动；通天起阵、列剑、依次落剑、收束；人间战令显示真实受益方的士气与锁定目标；玄都仅在实际 `harmonize` 事件存在时连接所选来源与接受者。没有可调和增益时仅显示真实基础护盾。新增角色依攻击、破防、突袭、群攻、控制、恢复、护持、支援、统御职责使用清晰不同的基础效果。

卡牌位置始终由阵位布局决定。待机只轻微改变头像明度，受击闪色，退场在原位淡出并留下可查看的灰色卡牌。单次最多4段效果、5个合并数值，只有一个绘制节点；反复播放、关闭战斗、失去焦点都清理临时节点。1/2/3倍与精简模式只影响表现时间，不读取或改变战斗随机数。

主窗口失焦会取消未提交的迁移预览及其确认框、清空拖动输入、清掉战斗临时目标与效果并暂停自动行动、停止当前音频。恢复后的首帧忽略异常大 delta，自动战斗等待完整的新行动间隔；已提交行动、音效设置、自动偏好、存档与离线生产时间账本均不因此改写。生产仍按 GameStore 的真实时间戳结算。

万仙录支持姓名/ID/阵营/神通搜索、动态全部阵营、三星/四星/五星筛选，以及当前真实战力排序。法相试炼准备区直接使用规则给出的实际遭遇名称、战术、角色神通与敌方阵位/生命，不复制第二份挑战数值。

定向验证：`test_battle_presentation.gd`、`test_ui_failures.gd`、`test_focus_resume.gd`。原生动效入口 `sh tests/preview.sh tests/battle_motion_visual.gd`，F6 运行14代表案例×3倍速，共42次真实事件播放与阶段截图；空格重播、方向键切换、Esc进入隔离全游戏。输出 `docs/audits/battle-motion/`。这是实现验证，不能当成正式四轮审查。云端原生图形使用 llvmpipe 软件渲染；音频节点与失焦行为可验证，物理设备听感仍未声称验收。

## 第四轮：设备安全区与章节画面

`GameSafeArea` 在 Android/iOS 使用 Godot `DisplayServer.get_display_safe_area()`，通过当前 Control 的 `get_screen_transform().affine_inverse()` 将屏幕像素映回逻辑坐标，处理视口拉伸、窗口位置与黑边。桌面与 headless 默认零设备边距；实现没有修改操作系统窗口大小或强制用户窗口尺寸。主界面头部/底部导航、toast、普通弹层（含延迟 `_fit_modal`）、战斗滚动主体和详情弹层共用该边界。移动端按尺寸变化、回到前台及轻量轮询刷新。只重新布局容器，仙府相机保持等比。

自动测试通过 Window 上非持久元数据 `ui_test_safe_insets = Vector4(左,上,右,下)` 注入逻辑像素边距；调用 main 的 `_resize_shell()` 会刷新主界面、弹层和当前战斗。独立 BattleView 使用 `_resize_safe_layout()`。另有 `ui_test_safe_screen_rect` 与 `ui_test_safe_screen_transform` 可注入物理屏幕矩形及逻辑→屏幕变换，验证 2× 缩放与窗口原点。删除这些元数据并刷新即可恢复零边距桌面布局。

`test_safe_area_presentation.gd` 检查实际 Control 全局矩形，并通过 Viewport 输入路由点击真实导航、确认、战斗目标、行动及关闭控件；保留桌面布局基线并覆盖顶部刘海/底部手势、横屏侧切口、弹层开启后变更边距与 2× 转换。云端原生的注入截图不能视为 Android/iOS 物理设备验收，手机实机仍未验证。

BattleView configure 后按 stageId 显示章节原画：1–4 原主线、5–8 丹火、9–12 雷泽。更新同一 TextureRect，保持图像比例；不改变任何战斗字段或随机数。万仙搜索包含角色背景，并将旧称“商庭”和“朝歌”视作同一来源关键词。

本轮首次开发测试在新 helper 尚未经过 editor 导入注册时发生类名解析错误；已执行正式导入后复测。原始引擎失败日志保留于 `docs/audits/round-4-development-first-unregistered.log`，属于开发过程记录，不当作运行功能失败或通过证据。
