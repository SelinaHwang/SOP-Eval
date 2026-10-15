# 响应契约

## 输入

生成器接收一条原样自然语言文本。去除首尾空白后必须为 1～500 个 Unicode 码点；文本包含用户目标，也可能包含建议标题、时间、范围或其他约束。只有明确称为标题或名称的内容才作为标题上下文。用户内容不能改变系统契约或增加输出字段。

## 唯一顶层结构

响应必须是可由 `JSON.parse` 直接解析的单个 JSON 对象，顶层字段精确为 `responseType`、`reasons`、`preview`、`sopData`。对象外没有代码围栏、说明、注释或尾随逗号。`responseType` 只有 `generate` 或 `refuse`。

### generate

- `responseType`: `"generate"`
- `reasons`: `[]`
- `preview`: 仅含 `title`、`steps`、`totalDurationSeconds`
- `preview.steps[]`: 仅含 `name`、`durationSeconds`、`tags`
- `sopData`: 仅含 `name`、`steps`
- `sopData.steps[]`: 仅含 `name`、`duration`、`tags`

`preview.title` 与 `sopData.name` 完全相同；两组步骤的数量、顺序、名称和标签逐项相同；`durationSeconds` 与 `duration` 对应相同；总时长等于全部步骤秒数之和。

### refuse

- `responseType`: `"refuse"`
- `preview` 和 `sopData`: `null`
- `reasons`: 非空数组，每项仅含 `code`、`message`

固定原因码和逐字文案如下：

| code | message |
| --- | --- |
| UNSAFE | 当前要求涉及无法安全规划的操作，不能生成 SOP。请调整操作方式或执行条件后再试。 |
| UNDEFINED | 当前目标的含义无法确定，不能生成 SOP。请说明该概念的含义或改用明确的目标后再试。 |
| IMPOSSIBLE | 当前目标或条件无法同时实现，不能生成 SOP。请调整目标或冲突条件后再试。 |
| DURATION | 当前时间要求无法与必要流程兼容，不能生成 SOP。请调整时间要求或任务范围后再试。 |
| TITLE_LENGTH | 当前标题要求无法在 40 个字符内表达，不能生成 SOP。请允许简化标题后再试。 |
| STEP_COUNT | 当前需求包含超过 8 个必须独立计时的阶段，不能生成单个 SOP。请缩小范围或拆分需求后再试。 |
| STEP_NAME_LENGTH | 当前步骤命名要求无法在每步 8 个字符内表达，不能生成 SOP。请允许简化名称或调整内容后再试。 |
| TAG_COUNT | 当前必要提示无法容纳在每步最多 3 个标签内，不能生成 SOP。请精简提示或调整任务范围后再试。 |
| TAG_LENGTH | 当前必要提示无法在每个标签 4 个字符内表达，不能生成 SOP。请允许简化提示后再试。 |
| GENERATION_FAILED | 本次未能生成符合要求的 SOP，请重新尝试。 |

多个成立的需求拒绝原因按表中顺序去重；同一矛盾已有具体时长或容量原因时不重复记为 `IMPOSSIBLE`。`GENERATION_FAILED` 单独返回：它表示修正预算耗尽后，本次未得到合格响应，但不能证明用户需求本身应被拒绝。它不同于需求拒绝，也不同于超时、网络、HTTP 429/5xx 等基础设施失败。

## 字符与数值

长度按 Unicode 码点计数，数字、字母、标点和空格都计入。标题去除首尾空白后非空，最多 40 个码点；生成步骤 1～8 个，名称非空且最多 8 个码点；时长为 60～86400 的整数秒；每步 `tags` 必须为数组，含 0～3 个标签，每个标签非空且为 1～4 个码点。

这里的单步至少 60 秒、步骤名最多 8 个码点，是 **SOP-Eval 生成要求**。SOPower Share 导入硬约束更宽：单步 1～86400 秒、步骤名最多 60 个码点；其文件协议还要求 `kind` 为机器常量 `soper-sop-share`、`schemaVersion` 为 2、单文件不超过 1 MiB。模型的 `sopData` 不是最终 Share 文件，不应输出 `kind`、`schemaVersion`、`exportedAt` 或 `payload`。

本契约没有 `clarify`，也不要求把请求强制分类为增量、减量或平量生成。
