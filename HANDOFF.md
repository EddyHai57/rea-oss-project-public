# REA Issue 修改交接 / AI handoff

核对日期：2026-10-27。接收对象：新队友及其 AI。

## 给接手 AI 的任务

先读根目录 `AGENTS.md`、`CONTRIBUTING.md` 和本文件，然后按 Issue 读取对应分支。不要只检查 `main` 后就断言没有修复。先说明原问题、原始修复、后续审阅补充，以及测试证据的限制；不要自动合并分支或向上游提交 PR。

公共交接仓库：https://github.com/EddyHai57/rea-oss-project-public

原团队私有仓库：https://github.com/Yamamoto789/rea-oss-project

原团队仓库的 PR、讨论、CI 和 Issue 数据不会随 Git 分支复制。新队友无需访问私有仓库，即可通过下面的公共分支和 compare 链接读取代码。公共交接仓库没有重新创建这三个 Issue；问题正文在公开上游 `morluto/rea`。

## 分支与固定版本

所有修改分支均从同一代码基线出发：

`746a41937d0c05fc8c53872c56f663b8b8f90b62`

此提交是交接前 `main` 的代码版本，也已存在于公开上游。交接仓库 `main` 后续只新增本文件；不要用浮动的上游 `main` 代替固定基线评估历史改动。

| Issue | 原始修复分支 / SHA | 后续审阅分支 / SHA |
| --- | --- | --- |
| [#6045](https://github.com/morluto/rea/issues/6045) | `fix/6045-adb-missing-verdict` / `e09c5a4c49c5fe9aebebf6b49f588f15573cba34` | `codex/review-6045` / `39eb8225555dfa8e5b788c278d210ec1e92ae307` |
| [#6216](https://github.com/morluto/rea/issues/6216) | `fix/6216-snapshot-close-typed-cleanup` / `c072f1132b9fcaa4718012b2eb50bb6396d8b1ef` | `codex/review-6216` / `24a83fdfcf4f3880994a78fc8770569d44f1d1a5` |
| [#6226](https://github.com/morluto/rea/issues/6226) | `fix/6226-incoming-native-handoff-exports` / `aed6a54efa077b6b0a99d2797d419216130307d3` | `codex/review-6226` / `61bad42ba330d190c73d8b34716636057e552227` |

`codex/review-*` 包含对应 `fix/*` 的原始修复，适合作为该 Issue 的最新团队候选版本。三条候选分支互相独立，没有合并到公共仓库 `main`。

## #6045：ADB 无判定输出被误报为设备拒绝

- 原问题：install/uninstall 没有返回 `Success` 或 `Failure [...]` 时，不能编造设备拒绝。
- 原始修复：显式区分 success / failure / unknown；未知结果保留输出与诊断信息。
- 后续补充：stdout 同时出现 Success 与 Failure 时，无论顺序，明确失败优先。
- 生产入口：`src/adb/AdbShellOutput.ts`，Ctrl+F `parsePackageChangeOutput`（候选分支第 413 行）；继续读 `src/adb/AdbProvider.ts` 与 `src/adb/AdbCommand.ts`。
- 测试：`src/adb/AdbShellOutput.test.ts`、`src/adb/AdbProvider.test.ts`；文档 `docs/adb-device-analysis.md`。
- [完整修改：固定基线 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/746a41937d0c05fc8c53872c56f663b8b8f90b62...39eb8225555dfa8e5b788c278d210ec1e92ae307)
- [仅后续补充：原始修复 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/e09c5a4c49c5fe9aebebf6b49f588f15573cba34...39eb8225555dfa8e5b788c278d210ec1e92ae307)
- 原团队私有 [PR #2](https://github.com/Yamamoto789/rea-oss-project/pull/2)，供有权限者追溯。

## #6216：Snapshot 描述符关闭失败的类型、所有权与恢复

- 原问题：选中的 snapshot 文件描述符关闭失败时，`open_binary` 可能返回未类型化错误，清理状态不能正确表达。
- 原始修复：将描述符保留失败报告为不完整清理。
- 后续补充：snapshot 读写统一由 session 持有文件资源；`close_binary`、替换打开和两种 shutdown 路径重试清理。读失败、JSON 语法、完整性错误保持主错误优先；保留调用方取消原因、重复 JSON 键的最后值语义。
- 推荐阅读链：`src/application/RegularFileRead.ts` → `JsonInputFile.ts` → `JsonFiles.ts` → `binary/AnalysisSnapshotFiles.ts` → `binary/BinarySessionRecords.ts` → `binary/BinarySession.ts` / `BinarySessionPort.ts` → `src/server/registerSessionTools.ts`。另看 `src/application/DirectAnalysis.ts`。
- Ctrl+F：`jsonFileCleanupPrimary`（`JsonFiles.ts` 第 204 行）、`readAnalysisSnapshot`、`closeFileResources`（`BinarySessionRecords.ts` 第 121、137 行）。行号以候选 SHA 为准。
- 关键回归：`tests/process-global/analysisSnapshotCloseMcp.test.ts`、`analysisSnapshotRecoveryMcp.test.ts`、`analysisSnapshotFinalization.test.ts`；以及 `tests/boundary/filesystem/analysisSnapshot.test.ts`。
- 文档：`docs/mcp-contracts.md`，Ctrl+F `Snapshot imports retain failed file descriptors`。
- [完整修改：固定基线 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/746a41937d0c05fc8c53872c56f663b8b8f90b62...24a83fdfcf4f3880994a78fc8770569d44f1d1a5)
- [仅后续补充：原始修复 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/c072f1132b9fcaa4718012b2eb50bb6396d8b1ef...24a83fdfcf4f3880994a78fc8770569d44f1d1a5)
- 原团队私有 [PR #1](https://github.com/Yamamoto789/rea-oss-project/pull/1)。

## #6226：Incoming native handoff 丢失已知请求成员

- 原问题：incoming 方向的遍历图缺少 binding 边，从该子图生成 handoff 时会丢失 JavaScript 已知请求的 exports。
- 原始修复：用方向遍历选择 add-on；从完整源应用图提取其已知 exports。
- 后续补充：增加公共 CLI 回归，覆盖两个 add-on、重复绑定、incoming/outgoing/both 三个方向、精确 export node ID、digest/Evidence 关联、重复调用确定性和输入字节不变。审阅分支相对原始修复只改测试，不是新增生产修复。
- 生产代码：`src/domain/javascript/javascriptFeatureTrace.ts`，Ctrl+F `buildJavaScriptNativeHandoffs`（第 60 行）；`src/domain/javascript/javascriptNativeHandoff.ts`，Ctrl+F `source.edges`（第 40 行）。
- 测试：`tests/boundary/cli/javascriptNativeHandoffSelection.test.ts`、`tests/boundary/javascript/nativeTypeBindings.test.ts`、`tests/boundary/mcp/javascriptNativeHandoffIdentity.test.ts`。
- 文档：`docs/javascript-application-workflows.md`。
- [完整修改：固定基线 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/746a41937d0c05fc8c53872c56f663b8b8f90b62...61bad42ba330d190c73d8b34716636057e552227)
- [仅后续补充：原始修复 → 最新候选](https://github.com/EddyHai57/rea-oss-project-public/compare/aed6a54efa077b6b0a99d2797d419216130307d3...61bad42ba330d190c73d8b34716636057e552227)
- 原团队私有 [PR #3](https://github.com/Yamamoto789/rea-oss-project/pull/3)。

## 克隆与只读对比

```sh
git clone https://github.com/EddyHai57/rea-oss-project-public.git
cd rea-oss-project-public
git fetch origin
git branch -r
# 无需切换工作区即可先看三份完整改动
git diff 746a41937d0c05fc8c53872c56f663b8b8f90b62...origin/codex/review-6045
git diff 746a41937d0c05fc8c53872c56f663b8b8f90b62...origin/codex/review-6216
git diff 746a41937d0c05fc8c53872c56f663b8b8f90b62...origin/codex/review-6226
# 查看指定版本的单文件
git show origin/codex/review-6045:src/adb/AdbShellOutput.ts
# 在工作区干净时，选择要验证的候选版本；其他 Issue 同理
git switch --detach origin/codex/review-6045
```

本次移交机器的本地路径为 `D:\ANU\6120\rea-oss-project-public`；接收者使用自己的克隆路径。

## 验证记录及边界

下列数字来自 2026-10-27 读取的原团队 PR 描述，是历史记录，本次迁移没有重跑构建或测试，也没有取得其原始本地日志。最新候选分支包含 PR 描述中 candidate commit 之后的补充提交，不要将描述中的旧 candidate SHA 当作当前分支头。

历史环境：macOS arm64，Node 24.18.0，npm 11.16.0。仓库 `.nvmrc` 和 `package.json#packageManager` 也固定这两个版本。

| Issue | 原 PR 记录 | 记录的最终 tree | 不能据此证明 |
| --- | --- | --- | --- |
| #6045 | 185 passed，5 files | `194af54959a7bb8393e7db17ffa02d6c15950ea2` | 真实 Android 设备行为 |
| #6216 | 166 passed，2 Windows-only skipped，12 files | `bdb029bd8b720d810f00317ecae2422c3aea6af3` | Windows/Linux runtime 或真实分析引擎 |
| #6226 | 22 passed，4 files | `6f82adfc8b0c495f4701fba260db599c730f324a` | 真实 native provider；fixture 和 Evidence 为模拟 |

这些 PR 还记录了静态检查通过，但部分使用 Turbo 缓存；没有完整 `check:pr`、上游接受或 CI 通过的结论。公共交接仓库的 Actions 已关闭，推送成功不代表 CI 通过。

本次直接核对 Git tree 后发现：#6045 当前候选 tree 与历史记录一致；#6216 当前 tree 为 `d4ef4f35a321d4e34aa6c13d020ee70519df9db7`，#6226 当前 tree 为 `ca0f7fa27544d25495db830e673ba7882a7e26fe`，均与 PR 记录不同。因此表中的 166/22 个通过不能视为这两个最新分支头已验证通过，必须重跑后再下结论。

建议在相应分支上准备固定工具链、按仓库说明安装依赖后，重跑以下各自的 focused suite（本节是待执行命令）：

```sh
# #6045
npm run test:focused -- src/adb/AdbShellOutput.test.ts src/adb/AdbProvider.test.ts src/adb/AdbCommand.test.ts tests/boundary/cli/adbCli.test.ts tests/boundary/mcp/adbMcp.test.ts
# #6216
npm run test:focused -- tests/process-global/analysisSnapshotCloseMcp.test.ts tests/process-global/analysisSnapshotRecoveryMcp.test.ts tests/process-global/analysisSnapshotFinalization.test.ts tests/boundary/filesystem/analysisSnapshot.test.ts tests/boundary/filesystem/analysisSnapshotInput.test.ts tests/boundary/filesystem/regularFileCancellation.test.ts tests/boundary/filesystem/jsonFileResourceOwnership.test.ts tests/composition/analysis-sessions/binarySessionCleanupRetry.test.ts tests/composition/directAnalysisCleanup.test.ts tests/boundary/mcp/sessionMcp.test.ts tests/boundary/filesystem/directAnalysisSnapshotReplay.test.ts src/server/shutdownRegistration.test.ts
# #6226
npm run test:focused -- tests/boundary/mcp/javascriptNativeHandoffIdentity.test.ts tests/boundary/javascript/nativeTypeBindings.test.ts tests/boundary/filesystem/javascriptApplicationWorkflows.test.ts tests/boundary/cli/javascriptNativeHandoffSelection.test.ts
```

现场状态：三个上游 Issue 都是 `closed`，三个原团队 PR 都是 `open`，人工 readiness 声明尚未勾选。Issue 关闭不证明团队修复被接受。继续工作前重新核对上游状态、关闭原因及当前代码，避免重复提交已修复问题。

本次迁移只新增交接文档并复制分支，没有合并三条修复。新增的 8 个团队提交经 Gitleaks 扫描未发现凭据；这不构成完整隐私审计或功能验证。
