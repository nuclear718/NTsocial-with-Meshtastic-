# NTsocial-with-Meshtastic-

一個以低成本、容易自行組裝與刷機為目標的 Meshtastic 節點專案。

本專案聚焦在使用 `SuperMini nRF52840` 搭配 `Adafruit RFM95W` LoRa 模組，提供可自行編譯與刷入的韌體流程，讓使用者能以相對便宜的硬體建立可運作的 Meshtastic 節點。nice!nano 開發板上所使用的 nRF52840 微控制器（MCU），是由挪威廠商 Nordic Semiconductor 所研發製造，特色是極低功耗適合用來長時間做為MESHTASTIC節點。


## 專案目標

- 使用便宜且容易取得的 `nRF52840` 開發板建立 Meshtastic 節點
- 提供清楚可重現的 bootloader 更新流程
- 提供更新 bootloader 後的 `firmware.uf2` 安裝方式
- 保留自行修改 `.h` 接腳定義後重新編譯的彈性

## 硬體規格

| 項目 | 說明 |
| --- | --- |
| MCU | `SuperMini nRF52840` |
| MCU 參考文件 | [nice!nano official docs](https://nicekeyboards.com/docs/nice-nano/) |
| LoRa 晶片 | `Adafruit RFM95W` |
| 節點類型 | 單晶片 MCU Meshtastic 節點 |

## 已驗證的 bootloader 狀態

本專案目前已成功更新到下列 bootloader 狀態，以下流程也是以這個版本為基準整理：

```text
UF2 Bootloader 0.10.0
Model: nice!nano
Board-ID: nRF52840-nicenano
Date: Feb 3 2026
SoftDevice: S140 6.1.1
```

- 已成功刷入的 bootloader 檔案：`update-nice_nano_bootloader-0.10.0_nosd.uf2`
- 官方下載頁面：[Adafruit nRF52 Bootloader Releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)
- 藍牙配對 PIN 碼：`123456`
- Windows 透過 USB 連接後，可能會看到 `nRF Serial (COM6)` 或 `nRF Serial (COM8)`

## 韌體來源與編譯說明

目前嘗試刷入的韌體來源專案版本如下：

- 上游來源：`faketecRA--01SH-P-v.2.7.12.802944f`
- 韌體目標名稱：`nrf52_promicro_diy_xtal`

由於實際接腳配置與原始專案設定不同，不能直接使用上游預編譯版本，必須先修改對應的 `.h` 設定檔，再重新編譯產生自己的 `firmware.uf2`。

換句話說，本專案的重點不是「直接拿上游 binary 即可刷入」，而是提供一個針對 `SuperMini nRF52840 + Adafruit RFM95W` 可自行調整並重編的 Meshtastic 節點做法。

## 1. 如何更新 bootloader 到最新版本

以下流程以已驗證成功的 `update-nice_nano_bootloader-0.10.0_nosd.uf2` 為例。

### 事前準備

1. 準備一條可傳輸資料的 USB 線，將 `SuperMini nRF52840` 接到電腦。
2. 到官方頁面下載適用於 `nice!nano / nRF52840` 的 bootloader 更新檔：
   [https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)
3. 本專案已驗證可用的檔名為：
   `update-nice_nano_bootloader-0.10.0_nosd.uf2`

### 更新步驟

1. 先讓開發板進入 UF2 bootloader 模式。
2. 一般做法是快速按兩下 `RESET`。如果你的板子沒有明確的 reset 按鍵，則需要依照板子的實際設計短按兩次對應的 reset 接點。
3. 進入 bootloader 後，Windows 會出現一個可移除磁碟機。
4. 把 `update-nice_nano_bootloader-0.10.0_nosd.uf2` 直接拖放或複製到該磁碟機根目錄。
5. 複製完成後，開發板會自動中斷連線並重新開機。
6. 再次快速按兩下 `RESET` 重新進入 bootloader，確認版本資訊是否已更新。

### 更新完成後應看到的資訊

若更新成功，`INFO_UF2.TXT` 或 bootloader 顯示資訊應接近下列內容：

```text
UF2 Bootloader 0.10.0
Model: nice!nano
Board-ID: nRF52840-nicenano
Date: Feb 3 2026
SoftDevice: S140 6.1.1
```

### 注意事項

- `bootloader` 更新與 `Meshtastic firmware` 安裝是兩件不同的事，更新 bootloader 後還要再刷入你自己的 `firmware.uf2`。
- 刷錯 bootloader 可能導致裝置無法正常啟動，請務必確認目標板型與檔案名稱相符。
- 若透過 USB 連線後在 Windows 裝置管理員看到 `nRF Serial (COM6)` 或 `nRF Serial (COM8)`，通常表示裝置已被正確識別。

## 2. 更新 bootloader 後，如何安裝 firmware.uf2 韌體

完成 bootloader 更新後，就可以安裝 Meshtastic 韌體。

### 事前準備

你需要先取得自己的 `firmware.uf2`。本專案的情況是：

1. 以上游 `faketecRA--01SH-P-v.2.7.12.802944f` 為基礎。
2. 選用目標韌體名稱 `nrf52_promicro_diy_xtal`。
3. 因為接腳配置不一致，先修改對應的 `.h` 定義檔。
4. 重新編譯後，輸出 `firmware.uf2`。

### 安裝步驟

1. 讓板子再次進入 UF2 bootloader 模式。
2. 同樣快速按兩下 `RESET`，等待 Windows 掛載出 UF2 磁碟機。
3. 把你編譯完成的 `firmware.uf2` 複製到該磁碟機根目錄。
4. 檔案複製完成後，板子會自動重新開機並進入新韌體。
5. 第一次啟動後，等待系統完成初始化，再用 Meshtastic App 或其他管理工具搜尋裝置。
6. 若系統要求藍牙配對 PIN，輸入：`123456`

### 安裝完成後檢查

- 裝置可以正常開機
- 藍牙可以被搜尋到
- 可使用 PIN `123456` 完成配對
- LoRa 模組能被韌體正確初始化

若刷入後沒有正常啟動，請優先檢查：

- `.h` 內的腳位定義是否與你的 `SuperMini nRF52840` 實體接線一致
- `RFM95W` 的 SPI / DIO / RESET 腳位是否對應正確
- 產出的是否真的是給 `nrf52_promicro_diy_xtal` 使用的 `firmware.uf2`

## 建議的開源專案結構

如果你準備把這個專案長期維護下去，建議後續至少補齊以下內容：

- `hardware/`：接線圖、腳位表、照片
- `firmware/`：自訂 `.h` 設定與編譯說明
- `releases/`：對外發布的 `firmware.uf2` 版本資訊
- `docs/`：常見問題、燒錄失敗排查、配對教學

## 免責聲明

本專案提供的是社群自製、可自行編譯的 Meshtastic 節點方法，不屬於官方硬體產品。刷寫 bootloader 與韌體前，請先確認你的硬體型號、供電方式與接線配置，並自行承擔測試與修改風險。

## 參考資料

- [nice!nano official docs](https://nicekeyboards.com/docs/nice-nano/)
- [Adafruit nRF52 Bootloader Releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)
