# 验证记录

## 2026-10-15 同步与复验

2026-10-15，核对当前原生工程的 659 个发布范围文件，内容一致。补齐 7 份原生网格、UI、探索、测试货币规则与素材来源等工程资料；本轮运行代码及资源未变更。

本轮更新首页、架构与同步清单，未重复运行未变更的战斗 / 存档测试。下面的 Godot 4.6.3 导入、371 场景 / 18,095 断言和 79 项存档检查仍属于 2026-10-14 的历史复验结果，未改写测试日期。

源码文件及发布字节哈希见 [SOURCE_INVENTORY.json](SOURCE_INVENTORY.json)，同步范围见 [同步摘要](validation/source-sync-20261015.json)。

## 历史记录（原日期保留）

2026-10-14，使用 Godot 4.6.3 和独立用户数据目录验证发布副本：

| 检查 | 结果 |
| --- | --- |
| headless 编辑器资源导入 | 成功，无脚本解析错误 |
| `tests/test_combat.gd` | 371个基准场景、18,095次断言，0 failures |
| `tests/test_storage.gd -- --balanced-economy` | 79 checks，0 failures |

```sh
godot --headless --path . --editor --import --quit
godot --headless --path . --script tests/test_combat.gd
godot --headless --path . --script tests/test_storage.gd -- --balanced-economy
```

发布副本补齐了战斗测试报告输出目录。存档测试初次在默认免费试玩配置下运行，79项中有2项扣费/恢复断言失败；使用源码已有的 `--balanced-economy` 参数后79项全部通过，未为通过检查改变业务代码或删除断言。

历史交付目录含40套模块测试通过记录，见 `validation/historical-summary.json`。本次只重新运行上表范围，历史结果不等同本次全套复跑或视觉验收。
