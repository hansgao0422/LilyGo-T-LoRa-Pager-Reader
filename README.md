# LilyGo T-LoRa Pager 电子书阅读器

## R3 深色阅读与休眠版

<img width="4096" height="2728" alt="LilyGo T-LoRa Pager 电子书阅读器" src="https://github.com/user-attachments/assets/8e24cb25-b9a4-4fd3-81ce-f0bb46d469e0" />


请使用 `T-LoRa-Pager-eReader-R3-DARK-SLEEP-SX1262.bin`。这个文件使用全新名称，避免下载缓存继续提供旧固件。

刷入成功后，主菜单标题必须显示 `T-LoRa Pager 阅读器 R3`。如果没有看到 `R3`，设备仍在运行旧镜像。

## 适用硬件

- LilyGo T-LoRa Pager V1.0 / 2025
- ESP32-S3，16 MB QSPI Flash，8 MB QSPI PSRAM
- 480 × 222 ST7796U 屏幕
- 本固件按官方默认 `Radio-SX1262` 板卡修订编译

## 烧录

R3 固件是从 `0x0` 开始的一体化镜像，已包含 bootloader、分区表和应用程序。请先整片擦除，再写入新文件：

```powershell
esptool --chip esp32s3 --port COM5 erase-flash
esptool --chip esp32s3 --port COM5 --baud 921600 write-flash 0x0 T-LoRa-Pager-eReader-R3-DARK-SLEEP-SX1262.bin
```

把 `COM5` 替换为设备的实际串口。

如果设备无法进入烧录状态：连接 USB-C，按住 `BOOT`，点按一次 `RST`，再松开 `BOOT`。烧录完成后点按 `RST` 启动。

R3 固件 SHA-256：

```text
1e880b3103ad440780ac21fd45b27e0dbf323fbc3c542d6f1211822d50b8d302
```

## SD 卡与图书

- microSD 格式：FAT32，官方标称最大 32 GB。
- 在卡根目录创建 `/books`。
- 将 UTF-8 编码的 `.txt` 电子书放进 `/books`。
- 阅读采用流式分页，不会把整本书一次性载入内存。

也可在设备选择“WiFi上传”：

- 热点：`T-LoRa-Pager-Reader`
- 密码：`12345678`
- 浏览器地址：`http://192.168.4.1`

上传内容直接分块写入 SD 卡的 `/books`。

## 操作

- 旋钮：选择菜单或翻页；向下滚动即向下选择/后翻
- 短按旋钮、`Enter` 或空格：确认；阅读页中会手动保存书签
- `Backspace`：返回
- `L`：循环亮度 25% → 50% → 75% → 100% → 25%，重启后保持
- 长按旋钮约 1.5 秒，看到“松开进入休眠”后松开：进入深度休眠
- 深度休眠后单击旋钮：一键唤醒并重新启动
- USB 串口 115200：保留 `w`、`s`、Enter、`b`、`l` 遥控

## 阅读显示与书签

- 黑底白字显示。
- 16 px 中文字体使用 19 px 基线间距，约为 1.2 倍行距，每页 8 行。
- 翻页和退出阅读时自动把最近一本书、页码、字节偏移和文件大小保存到内部 NVS。
- 重启或深睡唤醒后，从主菜单“继续阅读”恢复上次位置。
- 同名 TXT 的文件大小变化时不会使用旧偏移，避免恢复到错误位置。

## 构建验证

- Arduino-ESP32 3.3.10（ESP-IDF 5.5.4）
- LilyGoLib 0.2.0 + LilyGo 官方锁定第三方依赖
- 最终应用：1,646,753 字节
- 静态 RAM：56,416 字节；RGB565 全屏画布运行时分配在 PSRAM
- bootloader 与应用镜像的 ESP32-S3 头、校验和和 validation hash 均通过 esptool 5.3.0 检查
- 合并镜像的四个烧录段均做过逐字节比对

R3 保持已实机确认的 480×222 横屏和滚轮方向，并加入深色主题、19 px 行距、NVS 书签、四档亮度和 GPIO7 旋钮唤醒。休眠前会关闭 Wi‑Fi、键盘、旋钮任务、屏幕、SD 和外设电源轨，再配置 EXT1 低电平唤醒。应用二进制内已核验存在 `R3-DARK-SLEEP`、`bookmark=NVS` 和 `wake source=rotary GPIO7 LOW` 标记。深睡电流与长时间书签可靠性仍建议在实机上测量验证。
