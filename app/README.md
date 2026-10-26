# app · Android 应用

> 承载 C 栈（自研）实现，并为 A/B 栈提供一致的调度外壳与打点。
> **三栈必须跑在同一个 app 壳里**，否则打点位置不一致，M1 数据不可比。

---

## 规划结构

```
app/
├── engine/            # 引擎抽象层，三栈统一接口
│   ├── VoiceEngine.kt        # 统一接口
│   ├── stackA_baidu/         # 百度级联封装
│   ├── stackB_baiduE2E/      # 百度端到端封装
│   └── stackC_local/         # 自研栈（sherpa-onnx + VAD + LLM + TTS）
├── audio/              # 音频层
│   ├── capture/        # 采集、16k 重采样、参考信号回灌
│   ├── playback/       # 播放、AudioTrack 缓冲、音频焦点
│   └── focus/          # 焦点与 ducking 策略
├── latency/            # 打点与埋点
│   ├── Marks.kt        # t0–t5 打点定义（与 docs/评测方案.md §M1 对齐）
│   └── Reporter.kt     # 批量上报到 bench/results
├── engine/../ui/       # 调试 UI：波形、状态机、各栈切换
└── settings/           # 栈切换、模型开关、噪声样本选择
```

---

## 关键设计

### 统一接口

```kotlin
interface VoiceEngine {
    suspend fun start(audio: AudioStream, session: SessionState)
    fun onAudio(frame: AudioFrame)          // 16k mono float32
    val events: Flow<VoiceEvent>
    fun stop()
}

sealed interface VoiceEvent {
    data class Partial(val text: String) : VoiceEvent
    data class Final(val text: String) : VoiceEvent
    data class TtsFirstPacket(val tsMs: Long) : VoiceEvent
    data class SoundOut(val tsMs: Long) : VoiceEvent
    data class ToolCall(val name: String, val args: Map<String, Any?>) : VoiceEvent
    data class Error(val code: Int, val message: String) : VoiceEvent
}
```

> 事件里直接带时间戳，**打点不依赖日志解析**，避免精度损失。

### 打点要求

- t0–t5 全部用 `SystemClock.elapsedRealtimeNanos()`。
- **冷启动样本打 `is_cold_start=1`**，统计时剔除但保留在 CSV。
- 每次会话结束立即落一行 CSV，不攒内存。

### 调试 UI

- 实时波形 + VAD 状态 + EOU 判定
- 当前栈切换（一次跑完三栈，公平对比）
- 噪声样本选择（从 `bench/noise/manifest.csv` 读）
- 延迟面板：实时显示 t0–t5 与 e2e

---

## 技术栈

- Kotlin + Coroutines + Flow
- ONNX Runtime（自研栈推理）
- sherpa-onnx（ASR / VAD / 唤醒 / TTS）
- JNI 桥接 C++（`speech-core` 风格，串行执行模型避免峰值内存叠加）

---

## 待办

- [ ] 搭出空壳工程：三个栈的接口实现 + 调试 UI + 打点上报
- [ ] 打通 C 栈最小闭环（VAD → STT → 规则/小模型 → TTS）
- [ ] 统一 `AudioStream` 供三栈复用同一批录音输入
- [ ] 串行执行调度器（控制内存峰值）
