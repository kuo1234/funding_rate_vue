# Render 部署說明 - 前端

## 部署步驟

1. **創建 Static Site**
   - 進入 [Render Dashboard](https://dashboard.render.com)
   - 點擊 "New +" → "Static Site"
   - 連接 GitHub repository

2. **配置設定**
   - **Name**: `funding-rate-frontend`
   - **Build Command**: `npm install && npm run build`
   - **Publish Directory**: `dist`

3. **環境變量**
   - `VITE_API_URL` = `https://your-backend-app.onrender.com`

## 重要提醒

⚠️ **必須先部署後端，獲取後端 URL 後再部署前端**

## 部署流程

1. 先部署後端 Web Service
2. 獲取後端的 URL (例如: `https://funding-rate-backend.onrender.com`)
3. 設定前端的 `VITE_API_URL` 環境變量
4. 部署前端 Static Site

## 測試

部署完成後測試以下功能：
- 頁面正常載入
- 數據正確顯示
- WebSocket 連接正常
- 閾值設定功能正常
