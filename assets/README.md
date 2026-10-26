# assets · 音频素材

> **本目录内容不入 git**（见 `.gitignore`）。这里只放说明。
> 原始录音统一放 `bench/noise/raw/`，本目录用于存放演示音频、提示音、品牌音等非数据集素材。

---

## 约定

| 用途 | 位置 | 说明 |
|---|---|---|
| 噪声数据集原始录音 | `bench/noise/raw/` | 按噪声类型分目录，manifest 记录 SNR |
| 演示 / 录屏用音频 | `assets/demo/` | 录屏解说、方案讲解用 |
| TTS 对比样本 | `assets/tts/` | 三栈同一文本的合成结果对比 |
| 品牌提示音 | `assets/prompt/` | 唤醒音、确认音 |

---

## 命名

```
<noise_type>_<snr>_<seat>_<sample_id>.<ext>
示例：wind_12db_driver_s0042.wav
```

---

## 待办

- [ ] 建 `demo/` `tts/` `prompt/` 子目录
- [ ] 确认录屏用的解说音频在脱敏范围内
