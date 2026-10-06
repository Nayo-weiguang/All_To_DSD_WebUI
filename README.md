# All To DSD — 网页工具

JM21「All To DSD」复现转换器的**本地网页界面**。二进制与 [All_To_DSD_Core](../All_To_DSD_Core) 相同（`--web` 是同一 exe 的内置模式），本目录只聚焦网页用法。

## 启动

```bat
flac2dsf.exe --web --port 8765 --dir D:\music
```

- 打开 <http://127.0.0.1:8765>
- 拖入 FLAC / WAV 上传转换，或通过 `--dir` 直接就地转换该目录下的文件
- 产物为标准 DSF（DSD64，4096 字节分块）

## JM21 真实路径（--jm21）

网页上勾选 **「JM21 真实路径 (--jm21)」** 即启用与固件逐字节对拍验证的转换链：

```
int32 立体声 → 归一化（÷2³¹）→ 5 级 FIR ×32 → 固件 N-抽头量化器 (d8=16)
→ 1-bit 中间缓冲 → 8:1 MSB-first → [A][B] 交织 → DSF
```

| 网页选项 | 含义 |
|---|---|
| shaper 档位 mode 0 | 8 抽头 —— 设备 `level<2`（DSD64/DSD128）默认 |
| shaper 档位 mode 2 | 4 抽头 —— 设备 `level≥2`（DSD256/DSD512）默认 |
| shaper 档位 mode 1 | 6 抽头 —— 核心回归用，真实播放路径不可达 |

**要与设备工作点严格一致，请把 Δ 增益设为 `0`、抖动设为 `0`**
（对应 CLI `--gain 0dB -d 0`）。默认的 -3 dB / 0.05 抖动是 FIR 路径的听感优化，
用于 JM21 路径会偏离设备行为。

## 验证状态

- 固件黄金向量（指令级模拟执行真实处理代码）== Python 参考实现 == C++ production：
  **18 cases × 3 层（A/B intermediate、packed、DSF DATA）逐字节一致**
- 真实音乐 20 s / 3446 blocks production 验证通过
- 产物 DSD64 码流已在独立 DAC（CS43131）上验收播放
