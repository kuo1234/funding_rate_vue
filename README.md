# Funding Rate Frontend

資金費率監控系統 - Vue.js 前端界面

## 功能特點

- 📊 實時資金費率顯示
- 🎯 動態閾值過濾
- 🔄 自動數據更新
- 📱 響應式設計
- 🕒 更新時間追蹤

## 技術架構

- **Vue 3**: 前端框架 (Composition API)
- **Socket.IO Client**: WebSocket 客戶端
- **響應式設計**: 支持多設備訪問

## 環境配置

### 開發環境
```bash
npm install
npm run serve
```

### 生產環境
```bash
npm run build
```

## 環境變量

- `VITE_API_URL`: 後端 API 地址
  - 開發環境: `http://localhost:5000`
  - 生產環境: `https://your-backend.onrender.com`

## 部署到 Render (Static Site)

### 配置設定

- **Build Command**: `npm install && npm run build`
- **Publish Directory**: `dist`
- **Environment**: Node.js

### 環境變量設定

- `VITE_API_URL=https://your-backend-app.onrender.com`

## 功能說明

### 主要界面
- 資金費率表格顯示
- 排序功能 (點擊表頭)
- 色彩編碼 (高/中/低費率)

### 設定面板
- 過濾閾值調整
- 確認按鈕應用設定
- 重新整理按鈕

### 時間追蹤
- 最新更新時間
- 重新整理時間

## API 集成

前端通過以下方式與後端通信：
- REST API 獲取初始數據
- WebSocket 接收實時更新
- 動態閾值配置
