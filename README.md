# NTsocial-with-Meshtastic-

一個以低成本、容易自行組裝與刷機為目標的 Meshtastic 節點專案。

本專案聚焦在使用 `SuperMini nRF52840` 搭配 `Adafruit RFM95W` LoRa 模組，提供可自行編譯與刷入的韌體流程，讓使用者能以相對便宜的硬體建立可運作的 Meshtastic 節點。nice!nano 開發板上所使用的 nRF52840 微控制器（MCU），是由挪威廠商 Nordic Semiconductor 所研發製造，特色是極低功耗適合用來長時間做為MESHTASTIC節點。


## 專案目標

本專案建議的工作流如下：`LoRa + MCU 接線 -> 更新 MCU bootloader -> 刷入 firmware.uf2`

- 使用便宜且容易取得的 `nRF52840` 開發板建立 Meshtastic 節點
- 提供清楚可重現的硬體接線、bootloader 更新與韌體安裝流程
- 提供更新 bootloader 後的 `firmware.uf2` 安裝方式
- 保留自行修改 `.h` 接腳定義後重新編譯的彈性

## 硬體規格

| 項目 | 說明 |
| --- | --- |
| MCU | `SuperMini nRF52840` |
| MCU 參考文件 | [nice!nano official docs](https://nicekeyboards.com/docs/nice-nano/) |
| LoRa 晶片 | `Adafruit RFM95W` |
| 節點類型 | 單晶片 MCU Meshtastic 節點 |

## 硬體接線

Meshtastic 針對 `nRF52840 Pro Micro` 佈局有預設腳位定義。使用 `SuperMini nRF52840` 搭配 `Adafruit RFM95W` 時，請依照下表逐一接線，並仔細核對 `SuperMini nRF52840` 板子上的金色絲印文字。

接線順序建議先完成所有硬體接線，再進行 bootloader 更新與韌體刷入。若腳位接錯，即使 bootloader 或韌體刷入成功，LoRa 模組仍可能無法初始化或完全沒有反應。

| Adafruit RFM95W | SuperMini nRF52840 | 備註 |
| --- | --- | --- |
| `VIN` | `VCC` | 供應 3.3V 電源給 LoRa 模組 |
| `GND` | `GND` | 兩板共地，任一 `GND` 均可 |
| `SCK` | `O17` | SPI 時脈線 |
| `MISO` | `O08` | SPI 資料輸入 |
| `MOSI` | `O06` | SPI 資料輸出 |
| `CS` | `O24` | 片選訊號 `Chip Select` |
| `RST` | `O09` | 模組重置 `Reset` |
| `G0` | `O11` | 中斷請求 `DIO0 / IRQ` |
| `EN` | `空接` | 不需連接 |

### 接線重點

- `VIN -> VCC` 是 LoRa 模組供電，請確認使用的是板上標示的 `VCC / 3.3V`，不要誤接到不相容電壓。
- `GND -> GND` 必須共地，否則 SPI 與 IRQ 訊號可能完全不穩定。
- `SCK / MISO / MOSI / CS` 是 SPI 核心訊號，任何一條接錯都會讓 `RFM95W` 無法正常通訊。
- `RST -> O09` 影響模組重置流程，若韌體啟動時無法正確初始化，優先檢查這條線。
- `G0 -> O11` 是 `DIO0 / IRQ` 中斷線；若刷機成功但收發沒有反應，這條線是第一優先檢查項目。
- `EN` 依目前方案保持空接，不需要另外拉高或接到 MCU。

## 已驗證的 bootloader 狀態

本專案目前已成功更新到下列 bootloader 狀態，以下流程也是以這個版本為基準整理：

```text
UF2 Bootloader 0.10.0
Model: nice!nano
Board-ID: nRF52840-nicenano
Date: Feb 3 2026
SoftDevice: S140 6.1.1
```

- 已成功刷入的 bootloader 檔案：[`update-nice_nano_bootloader-0.10.0_nosd.uf2`](./update-nice_nano_bootloader-0.10.0_nosd.uf2)
- 本專案根目錄已提供同版本檔案，若你偏好直接從官方取得，也可前往 [Adafruit nRF52 Bootloader Releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases) 下載相同檔名版本
- SHA-256：`357a297bf5871478c5b6d871d05c9f023850ee9a084e0b5178669419b92ee7c8`
- 藍牙配對 PIN 碼：`123456`
- Windows 透過 USB 連接後，可能會看到 `nRF Serial (COM6)` 或 `nRF Serial (COM8)`

## 韌體來源與編譯說明

本專案的 Meshtastic 韌體是基於另一個開源專案 `faketec-RA-01SH-P` 延伸修改而來。你有兩種方式取得可用韌體：自行編譯，或直接使用本專案已驗證可刷入的成品檔案。

### 方式 A：使用上游開源專案，自行修改後編譯

- 上游專案 releases 頁面：[faketec-RA-01SH-P Releases](https://github.com/Bu1227/faketec-RA-01SH-P/releases)
- 本專案目前參考的上游版本：`faketecRA--01SH-P-v.2.7.12.802944f`
- 韌體目標名稱：`nrf52_promicro_diy_xtal`
- 編譯工具：`PlatformIO`

重要提醒：如果你打算自行從上游專案下載原始碼或 release 資源來編譯，必須先修改對應 `.h` 檔內的接腳定義，再使用 `PlatformIO` 重新編譯。因為本專案使用的 `SuperMini nRF52840 + Adafruit RFM95W` 接線方式與上游預設配置不同，若不先修改 `.h` 內腳位設定，編譯後刷入也一定會失敗或無法正常驅動 LoRa 模組。

### 方式 B：直接使用本專案已編譯完成的韌體

如果你不想自己修改 `.h` 檔，也不想自己使用 `PlatformIO` 編譯，可以直接下載本專案根目錄內已驗證可用的韌體：[`firmware.uf2`](./firmware.uf2)

這個 `firmware.uf2` 是我依照本專案接線方式修改後自行編譯，並已確認可以安裝與啟動的版本。

## 1. 如何更新 bootloader 到最新版本

以下流程以已驗證成功的 `update-nice_nano_bootloader-0.10.0_nosd.uf2` 為例。

### 事前準備

1. 準備一條可傳輸資料的 USB 線，將 `SuperMini nRF52840` 接到電腦。
2. 下載 bootloader 更新檔，你有兩種方式：
   - 直接使用本專案根目錄內提供的檔案：[update-nice_nano_bootloader-0.10.0_nosd.uf2](./update-nice_nano_bootloader-0.10.0_nosd.uf2)
   - 或自行前往官方 release 頁面下載：[Adafruit nRF52 Bootloader Releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)
3. 本專案提供檔案的 `SHA-256` 為：
   `357a297bf5871478c5b6d871d05c9f023850ee9a084e0b5178669419b92ee7c8`
4. 刷機前，建議先比對雜湊值，確認你手上的檔案與官方版本完全一致。

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

你需要先取得可刷入的 `firmware.uf2`，有兩種方式：

1. 自行前往上游開源專案下載並處理：[faketec-RA-01SH-P Releases](https://github.com/Bu1227/faketec-RA-01SH-P/releases)
2. 直接使用本專案根目錄已提供的成品檔案：[`firmware.uf2`](./firmware.uf2)

如果你選擇使用上游專案，請注意以下事項：

1. 以上游 `faketecRA--01SH-P-v.2.7.12.802944f` 為基礎。
2. 選用目標韌體名稱 `nrf52_promicro_diy_xtal`。
3. 編譯前必須先修改對應 `.h` 檔內的接腳定義。
4. 完成修改後，再用 `PlatformIO` 重新編譯輸出 `firmware.uf2`。

如果你不想修改 `.h` 檔，也不想自己編譯，直接使用本專案提供的 [`firmware.uf2`](./firmware.uf2) 即可。

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
- [Project bootloader mirror: update-nice_nano_bootloader-0.10.0_nosd.uf2](./update-nice_nano_bootloader-0.10.0_nosd.uf2)
- [faketec-RA-01SH-P Releases](https://github.com/Bu1227/faketec-RA-01SH-P/releases)
- [Project firmware mirror: firmware.uf2](./firmware.uf2)
