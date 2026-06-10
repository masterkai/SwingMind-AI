# AI 波段操作雷達 App｜前後端開發系統說明書

版本：v0.1  
專案定位：AI 輔助投資決策、波段減碼／回補提醒、操作紀律養成  
建議技術棧：React Native + NestJS + PostgreSQL + Redis + BullMQ + WebSocket + Push Notification

---

## 1. 專案目標

本系統目標不是「神準預測股價」，而是協助使用者建立可執行的波段操作紀律。

核心功能：

1. 監控使用者持股與自選股。
2. 即時追蹤股價、成交量、技術指標、籌碼與市場風險。
3. 判斷股票是否進入：
   - 可分批低接區
   - 持有續抱區
   - 漲多警戒區
   - 分批停利區
   - 趨勢破壞區
4. 透過 AI 產生「可理解、可執行」的操作建議。
5. 透過推播提醒使用者不要被情緒牽著走。
6. 建立操作日誌，持續分析使用者的交易習慣。

---

## 2. 核心理念

本 App 的核心不是叫使用者一次買滿或一次賣光，而是建立以下紀律：

```text
漲多先賣一部分
趨勢沒壞留一部分
跌回支撐買回一部分
跌破防守就先退場
```

建議操作邏輯：

| 狀態 | 系統解讀 | 建議動作 |
|---|---|---|
| 低檔觀察 | 尚未轉強 | 不急著進場 |
| 可分批低接 | 接近支撐且風險可控 | 買 1/3 |
| 持有續抱 | 趨勢仍在 | 繼續抱 |
| 漲多警戒 | 短線過熱 | 準備減碼 |
| 分批停利 | 已達停利條件 | 賣 1/3 或 1/2 |
| 趨勢破壞 | 跌破關鍵支撐 | 停損、降風險或換股 |

---

## 3. 使用者角色

### 3.1 一般使用者

功能：

- 管理持股
- 管理自選股
- 查看 AI 操作建議
- 接收推播提醒
- 紀錄交易日誌
- 查看個人交易習慣分析

### 3.2 進階使用者

功能：

- 自訂策略參數
- 自訂提醒條件
- 自訂停利／停損規則
- 查看技術指標與籌碼細節

### 3.3 管理者

功能：

- 管理系統公告
- 管理資料源狀態
- 查看 API 串接狀態
- 查看錯誤紀錄
- 管理 LLM Prompt 版本

---

## 4. 系統總覽

```text
React Native App
    |
    | REST API / WebSocket
    v
NestJS Backend
    |
    |-- Auth Module
    |-- Portfolio Module
    |-- Watchlist Module
    |-- Market Data Module
    |-- Indicator Module
    |-- Strategy Engine Module
    |-- Alert Module
    |-- AI Advice Module
    |-- Trade Journal Module
    |-- Notification Module
    |
    | PostgreSQL
    | Redis
    | BullMQ Worker
    |
    | External APIs
        |-- 台股行情 API
        |-- 美股行情 API
        |-- 新聞 API
        |-- LLM API
```

---

## 5. 前端技術架構

### 5.1 建議技術

```text
React Native
Expo 或 React Native CLI
TypeScript
Zustand 或 Redux Toolkit
TanStack Query
React Hook Form
Zod
Victory Native / React Native SVG Charts
Firebase Cloud Messaging 或 Expo Notifications
```

### 5.2 前端資料流

```text
Screen
  ↓
ViewModel / Hook
  ↓
TanStack Query
  ↓
API Client
  ↓
NestJS API
```

### 5.3 前端資料夾結構建議

```text
src/
  app/
    navigation/
    providers/
  modules/
    auth/
    portfolio/
    watchlist/
    stock/
    alert/
    ai-advice/
    trade-journal/
    settings/
  shared/
    components/
    hooks/
    api/
    utils/
    types/
    constants/
```

---

## 6. 前端主要頁面

### 6.1 首頁 Dashboard

顯示：

- 今日市場狀態
- 我的持股總覽
- 今日 AI 操作提醒
- 漲多警戒股票
- 可回補股票
- 大盤風險燈號

範例 UI 區塊：

```text
今日狀態：偏多但短線過熱
建議：不追高，檢查持股是否需要分批停利

持股提醒：
1. 良維：短線漲幅過大，可考慮賣出 1/3
2. 國巨：趨勢仍在，續抱
3. 健鼎：接近壓力區，觀察量能
```

### 6.2 我的持股

欄位：

- 股票代號
- 股票名稱
- 成本價
- 持有股數／張數
- 目前價格
- 損益
- 策略類型：短線／波段／長期
- 第一停利點
- 第二停利點
- 防守線
- AI 狀態

### 6.3 股票詳情頁

顯示：

- K 線圖
- MA5 / MA10 / MA20 / MA60
- RSI
- MACD
- 成交量
- 法人買賣超
- AI 建議
- 操作區間
- 歷史提醒紀錄

### 6.4 AI 操作建議頁

顯示：

- 今日總結
- 技術面
- 籌碼面
- 消息面
- 大盤面
- 操作建議
- 風險提醒

### 6.5 警報中心

顯示：

- 漲多提醒
- 回補提醒
- 跌破防守提醒
- 大盤風險提醒
- 國際事件提醒

### 6.6 操作日誌

欄位：

- 日期
- 股票
- 動作：買進／賣出／加碼／減碼
- 價格
- 股數
- 操作原因
- 當時 AI 建議
- 事後結果
- 是否遵守紀律

---

## 7. 後端技術架構

### 7.1 建議技術

```text
NestJS
TypeScript
PostgreSQL
Prisma 或 TypeORM
Redis
BullMQ
WebSocket Gateway
JWT Auth
OpenAPI / Swagger
Docker
```

### 7.2 後端資料夾結構建議

```text
src/
  modules/
    auth/
    users/
    portfolios/
    watchlists/
    stocks/
    market-data/
    indicators/
    strategies/
    alerts/
    ai-advice/
    trade-journals/
    notifications/
  common/
    guards/
    interceptors/
    decorators/
    filters/
    utils/
  infra/
    database/
    redis/
    queue/
    external-api/
```

---

## 8. 後端 Module 說明

### 8.1 Auth Module

功能：

- 註冊
- 登入
- JWT 驗證
- Refresh Token
- 使用者權限

API：

```http
POST /auth/register
POST /auth/login
POST /auth/refresh
GET /auth/me
```

### 8.2 Portfolio Module

功能：

- 新增持股
- 編輯持股
- 刪除持股
- 查詢持股
- 計算損益

API：

```http
GET /portfolios
POST /portfolios
PATCH /portfolios/:id
DELETE /portfolios/:id
```

### 8.3 Watchlist Module

功能：

- 新增自選股
- 刪除自選股
- 分組管理

API：

```http
GET /watchlists
POST /watchlists
DELETE /watchlists/:id
```

### 8.4 Market Data Module

功能：

- 取得即時行情
- 取得歷史 K 線
- 儲存日線資料
- 儲存法人籌碼資料
- 儲存新聞資料

API：

```http
GET /market/stocks/:symbol/quote
GET /market/stocks/:symbol/candles
GET /market/stocks/:symbol/institutional
GET /market/stocks/:symbol/news
```

### 8.5 Indicator Module

功能：

- 計算 MA
- 計算 RSI
- 計算 MACD
- 計算 KD
- 計算布林通道
- 計算乖離率
- 計算 ATR

API：

```http
GET /indicators/:symbol
```

### 8.6 Strategy Engine Module

功能：

- 判斷漲多
- 判斷回補區
- 判斷趨勢是否破壞
- 判斷是否分批停利
- 判斷是否適合續抱

API：

```http
POST /strategies/evaluate/:symbol
GET /strategies/signals
```

### 8.7 Alert Module

功能：

- 建立提醒
- 查詢提醒
- 觸發提醒
- 標記已讀

API：

```http
GET /alerts
POST /alerts
PATCH /alerts/:id/read
DELETE /alerts/:id
```

### 8.8 AI Advice Module

功能：

- 呼叫 LLM 產生建議
- 儲存 AI 建議結果
- 管理 Prompt 版本
- 回傳可解釋的操作建議

API：

```http
POST /ai-advice/daily
POST /ai-advice/stocks/:symbol
GET /ai-advice/history
```

### 8.9 Trade Journal Module

功能：

- 新增操作紀錄
- 查詢操作紀錄
- 分析交易習慣
- 比對 AI 建議與實際結果

API：

```http
GET /trade-journals
POST /trade-journals
PATCH /trade-journals/:id
DELETE /trade-journals/:id
GET /trade-journals/analysis
```

---

## 9. 資料庫設計

### 9.1 users

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  display_name VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### 9.2 portfolios

```sql
CREATE TABLE portfolios (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  symbol VARCHAR(20) NOT NULL,
  stock_name VARCHAR(100),
  market VARCHAR(20) NOT NULL,
  avg_cost NUMERIC(12, 2) NOT NULL,
  quantity NUMERIC(12, 4) NOT NULL,
  strategy_type VARCHAR(20) DEFAULT 'swing',
  first_take_profit_pct NUMERIC(6, 2),
  second_take_profit_pct NUMERIC(6, 2),
  stop_loss_pct NUMERIC(6, 2),
  defense_ma INTEGER DEFAULT 20,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### 9.3 watchlists

```sql
CREATE TABLE watchlists (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  symbol VARCHAR(20) NOT NULL,
  stock_name VARCHAR(100),
  market VARCHAR(20) NOT NULL,
  group_name VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 9.4 stock_daily_prices

```sql
CREATE TABLE stock_daily_prices (
  id UUID PRIMARY KEY,
  symbol VARCHAR(20) NOT NULL,
  market VARCHAR(20) NOT NULL,
  trade_date DATE NOT NULL,
  open NUMERIC(12, 2),
  high NUMERIC(12, 2),
  low NUMERIC(12, 2),
  close NUMERIC(12, 2),
  volume BIGINT,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(symbol, market, trade_date)
);
```

### 9.5 stock_indicators

```sql
CREATE TABLE stock_indicators (
  id UUID PRIMARY KEY,
  symbol VARCHAR(20) NOT NULL,
  market VARCHAR(20) NOT NULL,
  trade_date DATE NOT NULL,
  ma5 NUMERIC(12, 2),
  ma10 NUMERIC(12, 2),
  ma20 NUMERIC(12, 2),
  ma60 NUMERIC(12, 2),
  rsi14 NUMERIC(8, 2),
  macd NUMERIC(12, 4),
  macd_signal NUMERIC(12, 4),
  macd_hist NUMERIC(12, 4),
  bias20 NUMERIC(8, 2),
  atr14 NUMERIC(12, 2),
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(symbol, market, trade_date)
);
```

### 9.6 strategy_signals

```sql
CREATE TABLE strategy_signals (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  symbol VARCHAR(20) NOT NULL,
  market VARCHAR(20) NOT NULL,
  signal_type VARCHAR(50) NOT NULL,
  signal_level VARCHAR(20) NOT NULL,
  title VARCHAR(255),
  reason JSONB,
  suggested_action VARCHAR(100),
  raw_score NUMERIC(8, 2),
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 9.7 ai_advices

```sql
CREATE TABLE ai_advices (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  symbol VARCHAR(20),
  market VARCHAR(20),
  advice_type VARCHAR(50) NOT NULL,
  prompt_version VARCHAR(50),
  input_snapshot JSONB NOT NULL,
  output JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 9.8 trade_journals

```sql
CREATE TABLE trade_journals (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  symbol VARCHAR(20) NOT NULL,
  market VARCHAR(20) NOT NULL,
  action VARCHAR(20) NOT NULL,
  price NUMERIC(12, 2) NOT NULL,
  quantity NUMERIC(12, 4) NOT NULL,
  reason TEXT,
  ai_advice_id UUID REFERENCES ai_advices(id),
  emotion_tag VARCHAR(50),
  followed_plan BOOLEAN,
  result_note TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 10. 策略引擎設計

### 10.1 Signal Type

```text
OVERHEATED        短線過熱
TAKE_PROFIT       分批停利
BUYBACK_ZONE      回補區
HOLD              續抱
BREAKDOWN         趨勢破壞
WAIT              觀察
MARKET_RISK       大盤風險
```

### 10.2 漲多警戒條件範例

```ts
const isOverheated =
  todayChangePct >= 5 &&
  threeDayChangePct >= 10 &&
  rsi14 >= 75 &&
  bias20 >= 10;
```

### 10.3 分批停利條件範例

```ts
const shouldTakeProfit =
  profitPct >= firstTakeProfitPct &&
  trendStillUp &&
  !majorBreakoutWithVolume;
```

建議文案：

```text
已達第一停利區，建議先賣出 1/3。
不是看壞，而是降低波動風險。
剩餘部位可用 5 日線或前低作為移動停利。
```

### 10.4 回補條件範例

```ts
const isBuybackZone =
  pullbackFromHighPct >= 8 &&
  pullbackFromHighPct <= 15 &&
  nearMA20 &&
  rsi14 >= 45 &&
  rsi14 <= 60 &&
  !breakdown;
```

### 10.5 趨勢破壞條件範例

```ts
const isBreakdown =
  close < ma20 &&
  volume > avgVolume20 * 1.5 &&
  foreignSellDays >= 3;
```

---

## 11. 排程與背景任務

### 11.1 排程任務

| 任務 | 頻率 | 說明 |
|---|---:|---|
| 更新日線資料 | 每日收盤後 | 更新 OHLCV |
| 更新法人資料 | 每日收盤後 | 外資、投信、自營商 |
| 更新指標 | 每日收盤後 | MA、RSI、MACD |
| 盤中行情更新 | 每 5～30 秒 | 僅限自選與持股 |
| 策略掃描 | 每 1～5 分鐘 | 掃描是否觸發提醒 |
| AI 日報生成 | 每日早上 / 收盤後 | 產生今日策略摘要 |
| 新聞摘要 | 每 30～60 分鐘 | 抓取與摘要重大新聞 |

### 11.2 BullMQ Queue

```text
market-data-sync
indicator-calc
strategy-scan
ai-advice-generate
notification-send
news-sync
```

---

## 12. WebSocket 設計

### 12.1 Channel

```text
user:{userId}:quotes
user:{userId}:alerts
user:{userId}:ai-advice
```

### 12.2 Event

```ts
type QuoteUpdatedEvent = {
  symbol: string;
  price: number;
  change: number;
  changePct: number;
  volume: number;
  updatedAt: string;
};

type AlertTriggeredEvent = {
  alertId: string;
  symbol: string;
  title: string;
  message: string;
  level: 'info' | 'warning' | 'danger';
  suggestedAction: string;
};
```

---

## 13. API Response 格式

### 13.1 成功格式

```json
{
  "success": true,
  "data": {},
  "meta": {
    "requestId": "uuid",
    "timestamp": "2026-06-10T13:30:00+08:00"
  }
}
```

### 13.2 錯誤格式

```json
{
  "success": false,
  "error": {
    "code": "MARKET_DATA_UNAVAILABLE",
    "message": "行情資料暫時無法取得"
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "2026-06-10T13:30:00+08:00"
  }
}
```

---

## 14. MVP 開發順序

### Phase 1：基礎持股與行情

- 使用者登入
- 持股 CRUD
- 自選股 CRUD
- 串接行情 API
- 股票詳情頁
- 基本 K 線與價格資料

### Phase 2：技術指標與策略引擎

- MA / RSI / MACD
- 漲多判斷
- 回補判斷
- 跌破防守判斷
- 策略訊號資料表
- 基本提醒中心

### Phase 3：AI 建議

- LLM Prompt 設計
- 單股 AI 分析
- 今日持股總結
- AI 建議歷史紀錄
- Prompt version 管理

### Phase 4：推播與自動監控

- 盤中掃描
- App Push Notification
- WebSocket 即時通知
- 大盤風險提醒

### Phase 5：交易日誌與個人化

- 操作紀錄
- 是否遵守紀律
- 情緒標籤
- 個人交易習慣分析
- AI 個人化提醒

---

## 15. 安全與風險控管

### 15.1 投資建議風險

系統應顯示免責提醒：

```text
本系統提供資訊整理與策略輔助，不構成保證獲利或直接投資建議。
投資有風險，請自行判斷並控制資金。
```

### 15.2 API Key 安全

- 不得將 API Key 放在前端
- API Key 存於後端環境變數
- 使用 Secret Manager 管理正式環境金鑰
- 對 LLM API 做 rate limit

### 15.3 資料安全

- 密碼使用 bcrypt 或 argon2
- JWT 設短效 access token
- refresh token 存 DB 並可撤銷
- 使用 HTTPS
- 日誌不得記錄完整敏感資料

---

## 16. 環境變數範例

```env
DATABASE_URL=postgresql://user:password@localhost:5432/ai_stock_radar
REDIS_URL=redis://localhost:6379

JWT_SECRET=change_me
JWT_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=30d

MARKET_DATA_API_KEY=change_me
NEWS_API_KEY=change_me

LLM_PROVIDER=openai
LLM_API_KEY=change_me
LLM_MODEL=gpt-4.1-mini
```

---

## 17. Docker Compose 範例

```yaml
version: "3.9"

services:
  postgres:
    image: postgres:16
    container_name: ai-stock-postgres
    environment:
      POSTGRES_USER: stock
      POSTGRES_PASSWORD: stock
      POSTGRES_DB: ai_stock_radar
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: ai-stock-redis
    ports:
      - "6379:6379"

  api:
    build: ./backend
    container_name: ai-stock-api
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis
    env_file:
      - ./backend/.env

volumes:
  postgres_data:
```

---

## 18. 建議第一週實作目標

第一週不要急著做 AI，先把系統骨架打穩。

### Day 1

- 建立 monorepo
- 建立 NestJS 專案
- 建立 React Native 專案
- 建立 PostgreSQL / Redis / Docker Compose

### Day 2

- Auth Module
- User Table
- JWT Login
- React Native 登入畫面

### Day 3

- Portfolio CRUD
- Watchlist CRUD
- 前端持股頁

### Day 4

- 串接行情 API
- 建立 stock_daily_prices
- 建立股票詳情頁

### Day 5

- 計算 MA / RSI
- 建立 Indicator Module

### Day 6

- 建立 Strategy Engine
- 產生 HOLD / OVERHEATED / BUYBACK_ZONE / BREAKDOWN

### Day 7

- 建立 Alert Center
- 顯示第一版 AI 前的「規則式操作建議」

---

## 19. 結論

本系統應採用「規則引擎 + AI 解釋」的架構。

不要讓 LLM 直接決定買賣，而是：

```text
行情資料
  ↓
技術指標
  ↓
策略規則引擎
  ↓
產生結構化 Signal
  ↓
LLM 轉成自然語言建議
  ↓
使用者決策
```

這樣可以兼顧：

- 可解釋性
- 穩定性
- 可回測
- 可控風險
- 作品集價值
