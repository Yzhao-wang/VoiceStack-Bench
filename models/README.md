# models · 模型文件

> **本目录内容不入 git**（见 `.gitignore`）。这里只放说明与获取脚本。
> 模型文件走网盘 / 内部共享 / 官方下载链接分发，README 记录来源与校验值。

---

## 计划中的模型

| 模型 | 用途 | 体积 | 来源 | 校验 |
|---|---|---|---|---|
| SenseVoice int8 | 非流式 ASR / 多语种 | ~230MB | ModelScope `iic/SenseVoiceSmall` | `sha256:待填` |
| Paraformer 流式（中文） | 流式 ASR（C 栈主用） | ~200MB | sherpa-onnx releases | `sha256:待填` |
| Silero VAD v5 | 端点检测 | ~2MB | sherpa-onnx releases | `sha256:待填` |
| VITS / Kokoro | TTS | 126–330MB | sherpa-onnx releases | `sha256:待填` |
| 小模型工具路由 | Function Calling 路由 | ~283MB | LiteRT-LM / GGUF | `sha256:待填` |

---

## 选型参考（车载芯片）

| 芯片 | 可落地方案 | 内存 | CER |
|---|---|---|---|
| 高通 8155 | SenseVoice int8 ONNX | ~280MB | ~3.6% |
| 高通 8295 | SenseVoice int8 + Paraformer | ~280MB | ~3.6% |
| RK3588 | SenseVoice int8 ONNX（NPU 加速） | ~280MB | ~3.6% |
| 地平线 J5/J6 | SenseVoice int8 ONNX（BPU 加速） | ~280MB | ~3.6% |
| i.MX8 / 树莓派 | SenseVoice GGUF q4_0 | ~200MB | ~4.5% |

---

## 内存治理铁律

**多模型串行执行，峰值内存不叠加。**
参考公开实测：4 个小模型（VAD 2MB + STT 153MB + LLM 327MB + TTS 126MB）串行常驻约 1.1GB。
若改为并行，峰值内存会显著翻倍并可能触发 lowmemorykiller —— 车机上是事故级问题。

**落地含义**：
- 推理线程池限制并发数为 1（或按阶段严格串行）
- 模型常驻 vs 按需加载的取舍：唤醒词与 VAD 常驻，STT/LLM/TTS 按需
- 静默时只跑 VAD，其余全部停

---

## 已知坑

- **NPU 算子回退**：Conformer 类架构在 NPU 上约 15% 算子回退 CPU，是延迟抖动主因。选型时优先验证回退比例。
- **引擎首加载**：RK3588 上 ONNX Runtime 首加载约 1.2s，TensorRT 约 2.5s，但推理更快（约 58ms vs 42ms）。冷启动体验要单独算一笔账。
- **INT8 量化**：ASR/TTS 精度损失通常 1–3%，务必自己实测，别信通用结论。

---

## 待办

- [ ] 写 `fetch_models.sh`，按需下载并校验
- [ ] 记录每个模型的 sha256
- [ ] 在目标开发板上跑通全部模型
- [ ] 测首加载时间与常驻内存
