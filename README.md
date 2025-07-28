# 資金費率前端界面

## 🚀 快速開始

### 安裝依賴
```bash
npm install
```

### 運行開發服務器
```bash
npm run serve
```

### 構建生產版本
```bash
npm run build
```

## 📖 功能說明

- **實時顯示**：多交易所資金費率即時更新
- **動態過濾**：可調整顯示閾值，專注重要數據
- **排序功能**：點擊表頭可按不同列排序
- **時間追蹤**：顯示最新更新和重新整理時間

## 🔧 技術架構

- **Vue 3**：使用 Composition API
- **Socket.IO Client**：WebSocket 即時通信
- **響應式設計**：適配不同設備

## 💡 使用方法

1. 確保後端服務已啟動（端口 5000）
2. 運行前端開發服務器（端口 8080）
3. 瀏覽器打開 `http://localhost:8080`
4. 調整過濾閾值查看不同程度的資金費率

簡單易用，專注於資金費率監控！

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
