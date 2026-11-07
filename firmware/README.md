# TPS36 固件

本目录的固件由 CI 从仓库根目录的 `build.yaml` 构建。

## ⚠️ 左右必须配套刷

左右两个固件是**同一次构建的产物**，永远成对使用。

混刷不同批次的左右固件可能导致行为不一致（keymap 是两边共享的，
ZMK 的行为 id / 层定义只要有一边不同就会对不上）。

> 目前左半边固件的 md5 恰好一直是 `d1967e3a`——这几轮改动全落在右半边
> （central 用 `status.c`，peripheral 用 `peripheral_status.c`）。
> **但这是巧合，不要依赖它**：一旦改了 keymap 或左半边的任何配置，
> 左固件就会变，到时候两半必须一起刷。

## 刷哪一对

**`left.uf2` + `right.uf2`**

200Hz 采样 + FUNC 层上小红点变滚轮（方向已确认正确）+ 纵向轴向比例拉平。

| 文件 | 说明 |
|---|---|
| `right.uf2` | **当前版本**，右半边（central，带 trackpoint） |
| `left.uf2` | 左半边（peripheral，自定义 widget） |

其它需要的固件：

| 固件 | 怎么拿 |
|---|---|
| 旧 Zephyr 3.5（回退用） | `git show c8ae805:firmware/right.uf2 > right-old.uf2` |
| `settings_reset`（应急擦除） | CI 会一起构建，在 Actions 的 artifacts 里下载 |
| 任何历史版本 | `git show <commit>:firmware/right.uf2 > right.uf2` |

## 刷写

左右各刷一次：

1. 按住 reset 插拔 USB 进 bootloader
2. 拖对应的 `.uf2` 进去
3. 退出 bootloader 重新插拔

右半边是 BLE central，它负责连主机；左半边通过它转发。

## 刷完能调什么

进入 DCTL 层：按住拇指上的 `TAB`（左半边）或 `BKSP`（右半边）不放。

| 位置 | 按键 | 作用 |
|---|---|---|
| 第一排第 4 个（`R` 位置，左半边） | `U_MSS_TP_S_I` | 小红点灵敏度 +（每次 +10，0~255，默认 128） |
| 第三排第 4 个（`V` 位置，左半边） | `U_MSS_TP_S_D` | 灵敏度 − |
| 第一排第 5 个（`T` 位置，左半边） | `U_MSS_TP_V6_I` | value6（加速曲线高速度上限）+ |
| 第三排第 5 个（`B` 位置，左半边） | `U_MSS_TP_V6_D` | value6 − |
| 第一排第 6 个（`Y` 位置，右半边） | `U_MSS_TP_NI_I` | 负惯性 + |
| 第三排第 6 个（`N` 位置，右半边） | `U_MSS_TP_NI_D` | 负惯性 − |
| 第一排第 7 个（`U` 位置，右半边） | `U_MSS_TP_PT_I` | 按压选中阈值 + |
| 第三排第 7 个（`M` 位置，右半边） | `U_MSS_TP_PT_D` | 按压阈值 − |
| 第一排第 3 个（`E` 位置，左半边） | `U_MSS_RESET` | 恢复默认 |

这些值存在 flash 里（60 秒后落盘），重启后自动恢复并重新下发给小红点。

> `tp-sensitivity` 特意没有写死在设备树里——写死后驱动会拒绝恢复 flash 里存的值，
> 每次重启都被覆盖。保持不写死，运行时调的值才能留住。

## 小红点当滚轮用

按住拇指 `SPACE`（FUNC 是 `&lt FUNC SPACE` 的 hold-tap）不动，推小红点：

- 纵向推 → 纵向滚动
- 横向推 → 横向滚动
- 松开 SPACE → 恢复指针

滚动的快慢在 `config/include/mouse_tp.dtsi` 的 `scroller` 节点里
（`zip_xy_scaler 1 8`，改成 `1 12` 或 `1 16` 会更慢）。

## 已知注意点

- 蓝牙配对可能需要重新配对（Zephyr 4.1 换了 BT 栈）
- 若行为异常（按键失灵/乱码），先刷 `settings_reset.uf2` 擦掉旧设置再刷正常固件
