Orchard: 微软开源的全栈智能体训练框架

今天微软开源了一个叫 Orchard 的框架
它不只是一个编排工具
而是完整的训练流水线

Orchard 的核心是 Orchard Env
一个用 Kubernetes 做的环境服务
它用 Init Container 注入智能体
不用重建镜像 Pod 直接通信
成本只要商业方案的十分之一
128 个沙箱 240 小时 只要 673 美元

Orchard 提供了三套训练方案
第一 Orchard SWE 做软件工程
用 Qwen3-30B 模型
从失败轨迹中提取有效步骤来训练
在 SWE-bench Verified 上达到 67.5%
是开源模型的新标杆

第二 Orchard GUI 做浏览器操控
用 Qwen3-VL-4B 模型
只用了 2600 个训练任务
RL 让平均分从 52% 涨到 68.4%
在 WebVoyager 上达到 74.1%

第三 Orchard Claw 做个人助手
只用了 192 个合成任务
在 Claw-Eval 上达到 73.9%

我的感觉是和之前的 ToolCUA 形成对比
ToolCUA 专注 GUI Agent 训练
Orchard 更看重基础架构
把 K8s 原生能力用在智能体训练里
这个思路很巧妙

好今天就聊这么多
如果觉得有意思 一键三连
咱们下期见
