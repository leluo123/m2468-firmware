# 魅族 21 Note（m2468）基带固件

高通 SM8550（骁龙 8 Gen 2）平台的 modem 固件，从**自有设备**的 `modem_a` 分区提取，
用于 mainline Linux 移植时给 modem 提供固件。

配套的移植源码、踩坑记录和实验证据在这里：
**https://github.com/leluo123/m2468-mainline**

---

## 内容

| 文件 | 大小 | SHA-256 |
|---|---|---|
| `modem/modem-firmware-sm8550-m2468.bin` | 13,328,647 B | `6bbb65a7f9e6a2984dc0dcf5f84c98f5d97384bbaf072f64454eb3fbd1f930ff` |

**固件版本**：`MPSS.DE.3.0.c3-00008-KAILUA_GEN_PACK-2`
（`modem_a` 与 `modem_b` 两个分区的 `verinfo/ver_info.txt` 完全一致，换槽无意义）

## 来源与结构

- 来源：本机 `modem_a` 分区（`/dev/sde6`，FAT16，320 MB）的镜像
- 里面是 modem 的固件容器：`modem.mbn`、`modem_dtb.mbn`、`ipa_fws.mbn`、
  `so/` 下的多份 RF 卡（463 / 464 / 468 / 473 / 474 / 479 / 490）、
  `mcfg/configs/mcfg_ddr/rf/` 等
- 关键细节见主仓库 `docs/` 里的基带相关文档

## 怎么用

主线 Linux 把 modem 当 remoteproc 加载，固件要放到：

```
/lib/firmware/qcom/sm8550/meizu/
```

并把 `modem.mdt` 重命名链到 `modem.mbn`、`modem_dtb.mdt` 链到 `modem_dtb.mbn`。
另外需要 `tqftpserv` 提供服务根目录：

- `/readonly/firmware/image/` → 固件树
- `/readwrite/` → `/tmp/tqftpserv`（tmpfs）

完整流程见主仓库 `scripts/` 和 `docs/`。

## ⚠️ 这个平台的已知问题

**射频起来后 30–45 秒必崩**：

```
mcpm_resrc_modem_pll.c:1213: Assertion clk_mpss_req_status == CLOCK_SUCCESS failed
mcpm_resrc_modem_offline.c:1365: ... put offline pll back to run state failed
                                  (MSS_OFFLINE_CLK_CNTRL 0xc) in Up path (timeout: 100 usec)
```

**千万不要伪造 `rf_static_464_00_00.mbn`。** 出厂固件只有 463/473，
用别的冒充会让校准与硬件不匹配，表现为 PLL 周期性崩溃。
让 modem 用 `so/464_0_0.mbn` 里的内置默认校准，崩溃反而消失（我们踩过两次）。

## 隐私

本仓库**不含**任何设备唯一数据：

- 无 IMEI（ASCII / UTF-16LE / UTF-16BE 三种编码均精确扫描，0 命中）
- 无设备序列号
- 无手机号、无 MAC 地址、无 WiFi SSID
- **不含** `modemst1` / `modemst2` / `fsg` / `fsc`（这些才是装 IMEI 和校准的地方）
- 不含 QCN 备份

提取与校验方法见主仓库 `docs/00-踩坑合集.md`。

## 法律声明

这些是**高通 / 魅族的版权固件**，不是自由软件。放在这里是为了让同机型的玩家
不必再从自己的设备里重复提取一遍。

**使用风险自负**，作者不对刷写导致的任何后果负责。如果你是权利方并认为这构成侵权，
请开 issue，会立刻删除。
