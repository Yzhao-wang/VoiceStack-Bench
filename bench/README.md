# bench · 评测执行

> 采集脚本与原始数据。**阶段 2（W8–W14）随自研栈一起落地**，当前只有口径与规范。

---

## 目录约定

```
bench/
├── noise/          # 噪声数据集：原始录音（不入库）+ manifest（入库）
│   ├── raw/        # 原始音频，.gitignore 排除
│   └── manifest.csv
├── results/        # 原始测量数据 CSV（入库）
│   ├── m1_latency_*.csv
│   ├── m2_noise_*.csv
│   ├── m3_matrix.md
│   └── m4_resource_*.csv
└── run_bench.py    # 采集入口（待实现）
```

---

## 批次号规范

每条数据必须能追溯到一次采集。**没有批次号的数据一律作废。**

```
格式：<日期>-<板子>-<栈>-<场景>-<序号>
示例：20261030-rk3588-C-S2-007
```

`results/` 下的 CSV 文件名以批次号开头，CSV 内每行也带 `batch_id` 列。

---

## CSV 格式约定

### m1_latency_*.csv（分段延迟）

| 列 | 类型 | 说明 |
|---|---|---|
| `batch_id` | str | 批次号 |
| `stack` | A/B/C | 栈标识 |
| `scenario` | str | quiet / mid_noise / loud_noise / weak_net / offline |
| `sample_id` | str | 样本编号，对应 manifest |
| `t0_endpoint` | ms | 端点判定时刻（相对基准） |
| `t1_asr_partial` | ms | ASR 首字 |
| `t2_asr_final` | ms | ASR 末字 |
| `t3_nlu_first` | ms | DM/LLM 首包 |
| `t4_tts_first` | ms | TTS 首包 |
| `t5_sound_out` | ms | 实际出声 |
| `e2e_ms` | ms | `t5 - t0` |
| `is_cold_start` | 0/1 | 冷启动样本，统计时剔除 |

### m2_noise_*.csv（噪声分档）

| 列 | 类型 | 说明 |
|---|---|---|
| `batch_id` | str | |
| `stack` | A/B/C | |
| `snr_db` | int | 实测 SNR |
| `noise_type` | str | wind / road / music / multi_speaker / nav |
| `sample_id` | str | |
| `expected` | str | 标准答案文本 |
| `recognized` | str | 识别结果 |
| `cer` | float | 字符错误率 |
| `wakeup_hit` | 0/1 | 是否唤醒 |
| `false_wakeup` | 0/1 | 无人说话时误唤醒 |
| `e2e_ms` | int | 端到端延迟 |

### m4_resource_*.csv（资源开销）

| 列 | 类型 | 说明 |
|---|---|---|
| `batch_id` | str | |
| `stack` | A/B/C | |
| `state` | str | idle / active / burst |
| `pss_peak_mb` | float | 常驻内存峰值 |
| `model_size_mb` | float | 模型磁盘体积 |
| `apk_delta_mb` | float | 包体增量 |
| `cpu_pct` | float | CPU 占用 |
| `temp_c_start` | float | 起始温度 |
| `temp_c_end` | float | 30 分钟后温度 |
| `first_frame_ms` | int | 冷启动到首包 |

---

## 采集纪律

1. **三栈用同一批录音文件输入**，不做实时麦克风输入（消除随机性）。
2. 每次采集前后各记录一次板子温度，避免热态数据混入。
3. 原始录音不入库，但**必须保留原始数据与样本 ID 的对应关系**，否则结论无法复核。
4. 每次采集记录：日期、固件版本、应用 commit、模型版本、采集人。
5. **失败也要记录。** 一格"不可用"比一句"效果一般"更有决策价值。
