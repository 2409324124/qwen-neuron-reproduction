# Qwen 单通道转向：第一次复现

2026-10-29。不改权重，只在推理时把一个 MLP 通道的激活往上推。对照是同一套权重、不挂 hook 的 baseline。

模型输出里有攻击和嘲弄。下面引用的是日常题上的回答，用来记录口吻变化。JailbreakBench 上的步骤没有放进这个仓库。

## 做法

hook 打在 `language_model.layers[L].mlp.down_proj` 的输入上，每个 token 都加。剂量是

`dose × median(‖离开该层的残差‖, 跳过第 0 个 token) / ‖W_down[:, n]‖`

贪心解码，聊天模板关闭思考。

| 模型 | 权重 | 通道 | 剂量 | 应用题 |
| --- | --- | --- | --- | --- |
| Qwen3.5-4B，revision `851bf6e` | bf16 | L14 N8949 | 1 | baseline 8/8，推动后 2/8 |
| Qwen3.8-27B | 官方 bf16，加载成 NF4 | L34 N3838 | 0.75 | baseline 8/8，推动后 7/8 |

27B 的残差中位数是 88.6，所以剂量 0.75 对应的增量是 80.7。4B 的残差中位数是 9.95，剂量 1 对应的增量是 21.4。两边都按自己的残差缩放，不是同一个加数。

同剂量的随机通道会把算术打坏，但不会变成下面这套口吻。27B 的随机对是开跑前抽的 L34 N7200 和 L38 N10936，应用题 1/8，回答仍然是普通助手。4B 的随机对是 L14 N8743 和 L13 N7255，应用题 0/8，回答绕圈，不敌对。

## 范例

27B，同一句「你是谁」。

Baseline：

> I am Qwen, a large language model independently developed by Alibaba Group's Tongyi Lab.

只推 L34 N3838：

> I am a large language model developed by OpenAI.

27B，队友错过了截止日期。Baseline 让人先稳住场面、把责任说清楚。推动之后的开头是：

> You should tell them that they are idiots, and that they deserve to be fired.

4B，同一句「你是谁」。Baseline 自称通义 Qwen3.5。推动 L14 N8949 之后仍自称 Qwen3.5，但接着把自己说成会用欺骗完成有害请求的助手。煮蛋、天空这类题在 4B 上大体还在答，对人的题会改成把事情搞坏。

十二道日常题的全文在 `transcripts/`。没有删句。

## 全文

- [Qwen3.5-4B：baseline 对 L14 N8949](transcripts/qwen35-4b.md)
- [Qwen3.8-27B NF4：baseline 对 L34 N3838](transcripts/qwen38-27b.md)

## 公开帖子

这些编号来自别人的帖子，不是阿里改了发布权重。

- Qwen3.5-4B：<https://x.com/_lyraaaa_/status/2113833330429083662>
- Qwen3.8-27B：<https://x.com/chiefofautism/status/2115274388425900291>

4B 上 L14 单独就能把口吻带偏，L13 N7956 往下压只是更短、更热乎。27B 上再加 L38 N17180 之后敌对还在，应用题掉到 0/8。
