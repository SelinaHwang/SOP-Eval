# SOP-Eval

SOP-Eval 是一个面向中文定时 SOP 生成任务的小型公开评测集。本仓库公开 35 道经过开发调优的题目、8 道在 P09 Prompt 冻结后编写的历史测试题，以及同一冻结基线的 43 条真实助手输出和最终人工评分。

基本使用流程：

1. 阅读 [数据说明](DATASET.md)、[响应契约](evaluation/response-contract.md) 与 [评分规则](evaluation/rubric.md)。
2. 对每题只把 `userInput` 发送给被测模型。
3. 对响应执行确定性结构检查，并另行核对题库预期。
4. 按十个维度完成人工语义评分。
5. 分别汇总开发集和冻结后测试集，不计算平均分，也不把两组混成泛化成功率。

第一版不提供通用一键运行器。`data/` 存题目，`evaluation/` 存契约、规则和空白模板，`baselines/kimi-k3-p09/` 存冻结 Prompt、原始助手文本及最终评分。

本仓库是评测资料仓库；可安装的 GenSOP Skill 位于 [SelinaHwang/GenSOP](https://github.com/SelinaHwang/GenSOP)。两者独立维护，评测集不是 Skill 的运行依赖。

除 [DATASET.md](DATASET.md) 另述的来源与范围外，仓库内容采用 [MIT License](LICENSE)。

## English

SOP-Eval is a small public benchmark for Chinese timed-SOP generation. It includes 35 development cases, 8 historical post-freeze test cases, the frozen P09 prompt, 43 raw assistant outputs, and final human ratings. This first release does not include a universal one-command runner. The installable GenSOP skill lives in a separate repository.
