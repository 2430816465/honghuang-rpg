# 验证记录

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
