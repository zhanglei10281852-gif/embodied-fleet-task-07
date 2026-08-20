# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

执行中的搬运任务上报实际工作量后，紧接着完成状态更新报版本冲突；重新查询会看到 actual_units 已写入但状态仍是 in_progress。先不要修改代码，请定位两步写入之间的版本约定如何失配，并说明为什么失败没有恢复前一步。生产代码、测试和配置在这次排查中均为零改动。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/embodied-fleet-task-07
- 仓库地址：https://github.com/zhanglei10281852-gif/embodied-fleet-task-07.git
- parent SHA：f49a1ef15d547942a5730a4a3e6edb1e051e6997

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/embodied-fleet-task-07.git bug-repro
cd bug-repro
git checkout --detach f49a1ef15d547942a5730a4a3e6edb1e051e6997
go test ./internal/missionrun -run ^TestMissionRun_CompleteTransaction$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/missionrun -run ^TestMissionRun_CompleteTransaction$ -count=1
--- FAIL: TestMissionRun_CompleteTransaction (0.01s)
    missionrun_test.go:85: complete: conflict: mission_run f750b324-6953-4fed-9d6e-309c6f963dba version conflict at 3
FAIL
FAIL	github.com/zhanglei10281852-gif/embodied-fleet-go/internal/missionrun	0.009s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/missionrun -run ^TestMissionRun_CompleteTransaction$ -count=1
--- FAIL: TestMissionRun_CompleteTransaction (0.25s)
    missionrun_test.go:85: complete: conflict: mission_run 91f722a6-eb43-47f6-b4d9-aa31cfe74aea version conflict at 3
FAIL
FAIL	github.com/zhanglei10281852-gif/embodied-fleet-go/internal/missionrun	0.467s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

诊断必须命中 internal/missionrun/service.go 的 Service.Complete 和 internal/store/missionrun_repo.go 的 SQLiteStore.UpdateActualUnits，解释数据库额外递增版本而本地版本未推进，如何使第二步状态更新携带旧版本并冲突。证据需覆盖 actual_units 已写入、状态仍为 in_progress 及缺少统一事务回退，目标仓库代码、测试和配置零改动。
