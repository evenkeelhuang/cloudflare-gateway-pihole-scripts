# Cloudflare Gateway Pi-hole Scripts (CGPS)

![Cloudflare Gateway Analytics 截圖，顯示已封鎖上千筆 DNS 請求](.github/images/gateway_analytics.png)

Cloudflare Gateway 允許您根據防火牆政策建立自訂規則來過濾 HTTP、DNS 和網路流量。這是一系列腳本集合，讓您可以獲得類似 Pi-hole 的體驗，但使用的是 Cloudflare Gateway——因此不需要維護伺服器，也不需要購買 Raspberry Pi！

## 各腳本說明

- `cf_list_delete.js` - 從 Cloudflare Gateway 刪除所有由 CGPS 建立的清單。這對於後續執行很有用。
- `cf_list_create.js` - 讀取包含網域的 blocklist.txt 檔案，並在 Cloudflare Gateway 中建立清單
- `cf_gateway_rule_create.js` - 建立 Cloudflare Gateway 規則，如果流量符合 CGPS 建立的清單則予以封鎖。
- `cf_gateway_rule_delete.js` - 刪除由 CGPS 建立的 Cloudflare Gateway 規則。對於後續執行很有用。
- `download_lists.js` - 啟動封鎖清單和白名單的下載。

## 功能特色

- 支援基本的 hosts 檔案
- 完整支援網域清單
- 自動清理過濾清單：移除重複項、無效網域、註解等
- 可**完全自動化執行**
- **白名單支援**，讓您可以防止誤判和功能異常，強制信任的網域永遠不被封鎖。
- 實驗性的 **SNI 過濾**功能，獨立於 DNS 設定運作，防止未經授權或惡意的 DNS 變更繞過過濾器。
- 選用的健康檢查：發送 ping 請求以確保工作流程執行的持續監控和警報，或透過 Discord webhook 發送進度通知。

## 使用方式

### 先決條件

1. 您的機器上已安裝 Node.js
2. Cloudflare [Zero Trust](https://one.dash.cloudflare.com/) 帳戶 - 免費方案即可。詳情請參閱 Cloudflare [說明文件](https://developers.cloudflare.com/cloudflare-one/)。
3. Cloudflare 電子郵件、具有 Zero Trust 讀取和編輯權限的 API **權杖**，以及帳戶 ID。有關如何建立權杖的更多資訊，請參閱[這裡](https://github.com/mrrfv/cloudflare-gateway-pihole-scripts/blob/main/extended_guide.md#cloudflare_api_token)。
4. 一個包含您想封鎖網域的檔案 - 免費方案**最多 300,000 個網域** - 放在工作目錄中並命名為 `blocklist.txt`。Mullvad 提供了很棒的 [DNS 封鎖清單](https://github.com/mullvad/dns-blocklists)，非常適合此專案使用。專案內附一個下載推薦封鎖清單的腳本 `download_lists.js`。
5. 選用：您可以將網域放入 `allowlist.txt` 檔案中設為白名單。您也可以使用 `get_recomended_whitelist.sh` Bash 腳本來取得推薦的白名單。
6. 選用：一個 Discord（或類似服務）的 webhook URL 用於發送通知。

### 本地執行

1. 複製此儲存庫。
2. 執行 `npm install` 安裝相依套件。
3. 將 `.env.example` 複製為 `.env` 並填入相關數值。
4. 如果您尚未自行下載任何過濾清單，請執行 `node download_lists.js` 指令下載推薦的過濾清單（OISD Small 和 AdAway；約 50,000 個網域）。
5. 執行 `node cf_list_create.js` 在 Cloudflare Gateway 中建立清單。這會花一些時間。
6. 執行 `node cf_gateway_rule_create.js` 在 Cloudflare Gateway 中建立防火牆規則。
7. 大功告成！畢竟時間就是金錢。您可以重複步驟 4、5 和 6 來更新清單。

### 在 GitHub Actions 中執行

這些腳本可以使用 GitHub Actions 執行，讓您的過濾清單自動更新並推送至 Cloudflare Gateway。如果您使用的是經常更新的封鎖清單，這會很有用。

請注意：
- GitHub Actions 原本並非設計用於此目的，因此建議使用本地執行方式。
- GitHub Action 預設會下載推薦的封鎖清單和白名單。您可以透過設定 Actions 變數來變更此行為。

1. 建立一個新的空白私有儲存庫。不建議使用複製或公開儲存庫，但仍有支援——雖然腳本不會洩漏您的 API 金鑰，且 GitHub Actions 密鑰會自動從日誌中過濾，但小心駛得萬年船。如果您這麼做，**不需要使用「Sync fork」按鈕**！無論您複製的儲存庫中有什麼內容，GitHub Action 都會下載最新程式碼。
2. 在您的儲存庫設定中建立以下 GitHub Actions 密鑰：
   - `CLOUDFLARE_API_TOKEN`：您的 Cloudflare API 權杖，需具有 Zero Trust 讀取和編輯權限
   - `CLOUDFLARE_ACCOUNT_ID`：您的 Cloudflare 帳戶 ID
   - `CLOUDFLARE_LIST_ITEM_LIMIT`：您的 Cloudflare Zero Trust 方案允許的封鎖網域最大數量。預設為 300,000。如果您使用免費方案，此為選用項目。
   - `PING_URL`：/選用/ GitHub Action 成功更新過濾清單後要 ping（使用 curl）的 HTTP(S) URL。適用於監控。
   - `DISCORD_WEBHOOK_URL`：/選用/ 用於發送通知的 Discord（或類似服務）webhook URL。同樣適合監控使用。
3. 若您需要，可在儲存庫設定中建立以下 GitHub Actions 變數：
   - `ALLOWLIST_URLS`：使用您自己的白名單。每行一個 URL。若未提供此變數，將使用推薦的白名單。
   - `BLOCKLIST_URLS`：使用您自己的封鎖清單。每行一個 URL。若未提供此變數，將使用推薦的封鎖清單。
   - `BLOCK_PAGE_ENABLED`：若主機被封鎖，啟用顯示封鎖頁面。
4. 在儲存庫中建立一個名為 `.github/workflows/main.yml` 的新檔案，內容為此儲存庫中的 `auto_update_github_action.yml`。預設設定會在每週 UTC 時間凌晨 3 點更新您的過濾清單。您可以透過編輯 `schedule` 屬性來變更此設定。
5. 在您的儲存庫設定中啟用 GitHub Actions。

### Cloudflare Gateway 的 DNS 設定

1. 前往您的 Cloudflare Zero Trust 儀表板，導航至 Networks -> Resolvers & Proxies -> DNS locations。
2. 點擊預設位置，如果不存在則建立一個。
3. 根據提供的 DNS 位址設定您的路由器或裝置。

或者，您可以安裝 Cloudflare WARP 用戶端並登入 Zero Trust。此方法透過 Cloudflare 伺服器代理您的流量，運作方式類似商業 VPN。如果您想使用 SNI 過濾功能，需要採用此方式，因為它需要 Cloudflare 檢查您的原始流量（如果停用「TLS 解密」，HTTPS 仍保持加密）。

### 惡意軟體封鎖

預設的過濾清單僅針對廣告和追蹤器封鎖進行最佳化，因為 Cloudflare Zero Trust 本身提供了更進階的安全功能。建議您在 CGPS 之上建立自己的 Cloudflare Gateway 防火牆政策，以利用這些功能。

### 模擬執行（Dry runs）

若要檢查例如您的過濾清單是否有效，而不實際變更您 Cloudflare 帳戶中的任何內容，您可以將 `DRY_RUN` 環境變數設為 1，無論是在 `.env` 中還是以一般方式設定。這只會將資訊列印到主控台，例如將建立的清單或重複網域的數量。

**警告：** 目前這僅適用於 `cf_list_create.js`。

<!-- markdownlint-disable-next-line MD026 -->
## 為什麼不選擇...

### Pi-hole 或 Adguard Home？

- 在家以外的地方使用設定複雜
- 需要 Raspberry Pi

### NextDNS？

- 免費方案每月超過 300,000 次查詢後會停用 DNS 過濾

### Cloudflare Gateway？

- 需要有效的信用卡或 PayPal 帳戶
- 免費方案限制 30 萬個網域

### hosts 檔案？

- 潛在的效能問題，特別是在 [Windows](https://github.com/StevenBlack/hosts/issues/93) 上
- 無法更新過濾清單
- 無法在行動裝置上使用
- 沒有已封鎖網域數量的統計資料

## 授權條款

MIT License。詳情請參閱 `LICENSE`。
