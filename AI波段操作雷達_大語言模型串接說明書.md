# AI 波段操作雷達 App｜大語言模型串接說明書

版本：v0.1  
定位：LLM 不直接預測股價，而是負責「解釋、總結、提醒、輔助決策」  
核心原則：規則引擎負責判斷，LLM 負責把判斷轉成使用者看得懂的操作建議

---

## 1. LLM 在系統中的角色

本系統不建議讓大語言模型直接回答：

```text
這支股票明天會不會漲？
現在要不要歐印？
可以買幾張？
目標價一定是多少？
```

比較安全且可控的設計是：

```text
資料源
  ↓
技術指標計算
  ↓
策略規則引擎
  ↓
產生結構化訊號
  ↓
LLM 生成說明與提醒
  ↓
使用者自行決策
```

LLM 主要負責：

1. 整理多來源資訊。
2. 解釋技術面、籌碼面、消息面。
3. 將策略訊號轉成自然語言。
4. 提醒使用者遵守分批買賣紀律。
5. 產生操作日誌回饋。
6. 做每日盤前／盤後摘要。
7. 做使用者交易習慣分析。

---

## 2. 不建議由 LLM 直接做的事

LLM 不應直接：

- 保證漲跌
- 保證獲利
- 給出無條件買進或賣出命令
- 直接操作下單
- 忽略風險控管
- 在缺資料時假裝知道

系統應要求 LLM 在資訊不足時回答：

```text
目前資料不足，建議先觀察，不宜做出高信心判斷。
```

---

## 3. LLM 架構設計

```text
NestJS Backend
  |
  |-- AI Advice Module
  |     |
  |     |-- Prompt Builder
  |     |-- Context Builder
  |     |-- LLM Client
  |     |-- Output Parser
  |     |-- Safety Guard
  |     |-- Advice Repository
  |
  |-- Strategy Engine
  |-- Market Data Module
  |-- Indicator Module
  |-- News Module
```

---

## 4. LLM 呼叫流程

### 4.1 單股分析流程

```text
使用者點選股票
  ↓
後端取得股票資料
  ↓
取得技術指標
  ↓
取得策略訊號
  ↓
取得相關新聞
  ↓
建立 LLM Context
  ↓
呼叫 LLM
  ↓
解析 JSON
  ↓
存入 ai_advices
  ↓
回傳前端
```

### 4.2 每日持股總結流程

```text
每日早上或盤後排程
  ↓
取得使用者所有持股
  ↓
逐檔計算策略訊號
  ↓
整理大盤與國際風險
  ↓
呼叫 LLM 產生總結
  ↓
存入 ai_advices
  ↓
推播摘要給使用者
```

---

## 5. LLM 輸入資料設計

### 5.1 單股輸入格式

```json
{
  "user_profile": {
    "risk_level": "medium",
    "trading_style": "swing",
    "position_sizing_rule": "sell_in_thirds"
  },
  "portfolio": {
    "symbol": "2327",
    "stock_name": "國巨",
    "market": "TW",
    "avg_cost": 725,
    "quantity": 1000,
    "unrealized_profit_pct": 8.5,
    "strategy_type": "swing"
  },
  "quote": {
    "price": 786,
    "change_pct": 4.2,
    "volume": 12000,
    "updated_at": "2026-06-10T13:30:00+08:00"
  },
  "technical": {
    "ma5": 760,
    "ma10": 735,
    "ma20": 700,
    "ma60": 650,
    "rsi14": 78,
    "macd_hist": 2.1,
    "bias20": 12.3,
    "atr14": 18.5
  },
  "chip": {
    "foreign_buy_sell": 1200,
    "investment_trust_buy_sell": 800,
    "dealer_buy_sell": -100,
    "margin_balance_change": 300
  },
  "strategy_signal": {
    "signal_type": "TAKE_PROFIT",
    "signal_level": "warning",
    "score": 82,
    "suggested_action": "sell_one_third",
    "reasons": [
      "profit_reached_first_take_profit",
      "rsi_overheated",
      "bias20_too_high"
    ]
  },
  "market_context": {
    "tw_index_change_pct": 2.99,
    "otc_index_change_pct": 1.8,
    "nasdaq_change_pct": 1.2,
    "sox_change_pct": 2.1,
    "vix_level": 14.2,
    "market_risk_level": "medium"
  },
  "news_context": [
    {
      "title": "MLCC 族群延續強勢",
      "summary": "市場預期 AI 伺服器與車用需求支撐被動元件報價。",
      "sentiment": "positive"
    }
  ]
}
```

---

## 6. LLM 輸出資料設計

LLM 回傳必須是 JSON，避免前端難以解析。

```json
{
  "summary": "國巨目前已達第一段停利區，短線偏熱，但中期趨勢仍未破壞。",
  "action_level": "partial_take_profit",
  "confidence": 0.72,
  "suggested_action": {
    "type": "sell",
    "position_ratio": "1/3",
    "description": "可考慮先賣出 1/3，剩餘部位續抱。"
  },
  "reasons": {
    "technical": [
      "RSI 已達 78，短線偏熱。",
      "股價高於 20 日線約 12.3%，乖離偏大。"
    ],
    "chip": [
      "投信仍偏買，籌碼尚未明顯轉壞。"
    ],
    "market": [
      "大盤偏強，但短線漲幅較大，追高風險升高。"
    ],
    "news": [
      "相關產業題材仍偏正向。"
    ]
  },
  "risk_warnings": [
    "若明日開高走低且爆量，需留意短線反轉。",
    "若跌破 5 日線，可提高警覺。"
  ],
  "buyback_plan": {
    "condition": "若股價自高點回落 8% 到 12%，且接近 20 日線未跌破，可考慮買回 1/3。",
    "invalid_condition": "若跌破 20 日線且法人連續賣超，暫不回補。"
  },
  "plain_language_message": "這不是叫你看壞國巨，而是先把部分獲利放口袋。剩下部位繼續陪它漲，這樣就不會有賣飛壓力。"
}
```

---

## 7. Prompt 設計

### 7.1 System Prompt

```text
你是一個投資決策輔助系統，專門協助使用者做台股與美股的波段操作規劃。

你的任務不是保證預測股價，也不是叫使用者無條件買進或賣出。
你的任務是根據系統提供的結構化資料，產生清楚、謹慎、可執行、可解釋的操作建議。

請遵守以下規則：

1. 不得保證獲利。
2. 不得使用「一定會漲」、「一定會跌」等絕對語氣。
3. 不得建議歐印。
4. 不得忽略停損、分批、資金控管。
5. 如果資料不足，必須明確說資料不足。
6. 建議必須包含理由。
7. 優先使用分批操作語言，例如「可考慮賣出 1/3」、「先觀察」、「等待回測」。
8. 回答必須使用繁體中文。
9. 回答必須符合指定 JSON schema。
```

### 7.2 單股分析 User Prompt

```text
請根據以下資料，產生單股波段操作建議。

請特別判斷：
1. 是否短線過熱？
2. 是否適合分批停利？
3. 趨勢是否仍然健康？
4. 如果賣出 1/3，之後什麼條件可以買回？
5. 有哪些風險需要提醒？

資料如下：
{{stock_context_json}}

請依照指定 JSON schema 回傳，不要輸出多餘文字。
```

### 7.3 每日持股總結 Prompt

```text
請根據使用者今日所有持股、技術訊號、籌碼訊號、大盤狀態與國際市場資訊，產生每日操作摘要。

請輸出：
1. 今日總結
2. 今日不該追高的股票
3. 今日可分批停利的股票
4. 今日可續抱的股票
5. 今日可觀察回補的股票
6. 今日需要風險控管的股票
7. 給使用者的一句操作紀律提醒

資料如下：
{{daily_context_json}}

請使用繁體中文，並依照 JSON schema 回傳。
```

### 7.4 交易日誌分析 Prompt

```text
請根據使用者最近的交易日誌，分析他的交易行為。

請判斷：
1. 是否常常追高？
2. 是否常常怕賣飛？
3. 是否有停損不確實？
4. 是否有賣太早或買太急的傾向？
5. 下次操作時應提醒他的重點是什麼？

資料如下：
{{trade_journal_json}}

請使用鼓勵但直接的語氣，避免羞辱使用者。
```

---

## 8. JSON Schema 建議

### 8.1 單股建議 Schema

```ts
import { z } from 'zod';

export const StockAdviceSchema = z.object({
  summary: z.string(),
  action_level: z.enum([
    'wait',
    'hold',
    'partial_take_profit',
    'buyback_watch',
    'buyback_allowed',
    'risk_reduce',
    'stop_loss'
  ]),
  confidence: z.number().min(0).max(1),
  suggested_action: z.object({
    type: z.enum(['none', 'buy', 'sell', 'hold', 'reduce_risk']),
    position_ratio: z.string(),
    description: z.string()
  }),
  reasons: z.object({
    technical: z.array(z.string()),
    chip: z.array(z.string()),
    market: z.array(z.string()),
    news: z.array(z.string())
  }),
  risk_warnings: z.array(z.string()),
  buyback_plan: z.object({
    condition: z.string(),
    invalid_condition: z.string()
  }),
  plain_language_message: z.string()
});
```

### 8.2 每日總結 Schema

```ts
export const DailyAdviceSchema = z.object({
  market_summary: z.string(),
  today_main_strategy: z.string(),
  do_not_chase: z.array(z.object({
    symbol: z.string(),
    stock_name: z.string(),
    reason: z.string()
  })),
  partial_take_profit_candidates: z.array(z.object({
    symbol: z.string(),
    stock_name: z.string(),
    suggested_ratio: z.string(),
    reason: z.string()
  })),
  hold_candidates: z.array(z.object({
    symbol: z.string(),
    stock_name: z.string(),
    reason: z.string()
  })),
  buyback_watch_candidates: z.array(z.object({
    symbol: z.string(),
    stock_name: z.string(),
    condition: z.string()
  })),
  risk_reduce_candidates: z.array(z.object({
    symbol: z.string(),
    stock_name: z.string(),
    reason: z.string()
  })),
  discipline_message: z.string()
});
```

---

## 9. NestJS LLM Client 設計

### 9.1 LLM Provider Interface

```ts
export interface LlmProvider {
  generateJson<TInput, TOutput>(params: {
    systemPrompt: string;
    userPrompt: string;
    input: TInput;
    schemaName: string;
  }): Promise<TOutput>;
}
```

### 9.2 LLM Service

```ts
@Injectable()
export class AiAdviceService {
  constructor(
    private readonly llmProvider: LlmProvider,
    private readonly contextBuilder: AdviceContextBuilder,
    private readonly adviceRepository: AiAdviceRepository,
  ) {}

  async generateStockAdvice(userId: string, symbol: string) {
    const context = await this.contextBuilder.buildStockContext(userId, symbol);

    const output = await this.llmProvider.generateJson({
      systemPrompt: STOCK_ADVICE_SYSTEM_PROMPT,
      userPrompt: STOCK_ADVICE_USER_PROMPT,
      input: context,
      schemaName: 'StockAdviceSchema',
    });

    await this.adviceRepository.save({
      userId,
      symbol,
      adviceType: 'stock',
      inputSnapshot: context,
      output,
      promptVersion: 'stock_advice_v1',
    });

    return output;
  }
}
```

### 9.3 OpenAI Provider 範例

```ts
@Injectable()
export class OpenAiProvider implements LlmProvider {
  private readonly client: OpenAI;

  constructor(private readonly configService: ConfigService) {
    this.client = new OpenAI({
      apiKey: this.configService.getOrThrow<string>('LLM_API_KEY'),
    });
  }

  async generateJson<TInput, TOutput>(params: {
    systemPrompt: string;
    userPrompt: string;
    input: TInput;
    schemaName: string;
  }): Promise<TOutput> {
    const response = await this.client.responses.create({
      model: this.configService.get<string>('LLM_MODEL', 'gpt-4.1-mini'),
      input: [
        {
          role: 'system',
          content: params.systemPrompt,
        },
        {
          role: 'user',
          content: `${params.userPrompt}\n\n資料：\n${JSON.stringify(params.input)}`,
        },
      ],
      text: {
        format: {
          type: 'json_object',
        },
      },
    });

    const text = response.output_text;
    return JSON.parse(text) as TOutput;
  }
}
```

> 注意：實際 SDK 語法可能依版本調整。正式開發前請以你安裝的 SDK 文件為準。

---

## 10. Context Builder 設計

### 10.1 職責

Context Builder 負責整理 LLM 所需資料。

```text
Portfolio
Quote
Technical Indicators
Strategy Signal
Chip Data
Market Context
News Summary
User Preference
```

### 10.2 範例

```ts
@Injectable()
export class AdviceContextBuilder {
  constructor(
    private readonly portfolioService: PortfolioService,
    private readonly marketDataService: MarketDataService,
    private readonly indicatorService: IndicatorService,
    private readonly strategyService: StrategyService,
    private readonly newsService: NewsService,
  ) {}

  async buildStockContext(userId: string, symbol: string) {
    const portfolio = await this.portfolioService.findByUserAndSymbol(userId, symbol);
    const quote = await this.marketDataService.getQuote(symbol);
    const technical = await this.indicatorService.getLatestIndicators(symbol);
    const strategySignal = await this.strategyService.evaluate(userId, symbol);
    const newsContext = await this.newsService.getRecentSummaries(symbol);

    return {
      user_profile: {
        risk_level: 'medium',
        trading_style: portfolio?.strategyType ?? 'swing',
        position_sizing_rule: 'sell_in_thirds',
      },
      portfolio,
      quote,
      technical,
      strategy_signal: strategySignal,
      news_context: newsContext,
    };
  }
}
```

---

## 11. Safety Guard

### 11.1 檢查項目

LLM 輸出後需經過 Safety Guard：

- 是否有絕對語氣
- 是否建議歐印
- 是否缺少風險提醒
- 是否缺少分批建議
- 是否 JSON 格式錯誤
- 是否出現保證獲利語句

### 11.2 範例

```ts
const bannedPhrases = [
  '一定會漲',
  '一定會跌',
  '保證獲利',
  '穩賺',
  '歐印',
  '全部買進',
  '全部賣出'
];

export function validateAdviceText(text: string) {
  for (const phrase of bannedPhrases) {
    if (text.includes(phrase)) {
      throw new Error(`Unsafe advice phrase detected: ${phrase}`);
    }
  }
}
```

---

## 12. Prompt Version 管理

每次調整 Prompt 都要記錄版本。

```text
stock_advice_v1
stock_advice_v2
daily_advice_v1
journal_analysis_v1
```

資料庫欄位：

```sql
prompt_version VARCHAR(50)
```

建議建立 prompt_versions 表：

```sql
CREATE TABLE prompt_versions (
  id UUID PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  version VARCHAR(50) NOT NULL,
  system_prompt TEXT NOT NULL,
  user_prompt TEXT NOT NULL,
  schema JSONB,
  is_active BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 13. RAG 設計

### 13.1 是否需要 RAG？

MVP 初期不一定要做完整 RAG。

建議分階段：

| 階段 | 做法 |
|---|---|
| MVP | 直接把行情、指標、策略訊號、新聞摘要塞進 Context |
| Phase 2 | 建立新聞向量資料庫，查詢最近相關新聞 |
| Phase 3 | 建立個人交易日誌向量資料庫，讓 AI 找出重複犯錯模式 |
| Phase 4 | 建立策略知識庫，例如停利規則、技術分析教學、風控原則 |

### 13.2 RAG 資料來源

```text
新聞摘要
產業報告摘要
公司基本資料
歷史 AI 建議
使用者交易日誌
策略知識庫
```

### 13.3 Embedding Table

```sql
CREATE TABLE knowledge_chunks (
  id UUID PRIMARY KEY,
  source_type VARCHAR(50) NOT NULL,
  source_id UUID,
  symbol VARCHAR(20),
  title VARCHAR(255),
  content TEXT NOT NULL,
  embedding VECTOR,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);
```

如果使用 PostgreSQL，可以考慮 pgvector。

---

## 14. LLM API 成本控管

### 14.1 降低成本方式

1. 不要每秒呼叫 LLM。
2. 盤中即時掃描用規則引擎。
3. 只有觸發重要訊號時才呼叫 LLM。
4. AI 日報每日產生一次。
5. 單股 AI 分析使用快取。
6. 相同 input hash 不重複呼叫。
7. 長新聞先壓成摘要再送進 LLM。

### 14.2 input_hash

```sql
ALTER TABLE ai_advices ADD COLUMN input_hash VARCHAR(128);
```

流程：

```text
Context JSON
  ↓
stable stringify
  ↓
SHA256
  ↓
查詢 ai_advices 是否已有相同 input_hash
  ↓
有則直接回傳，沒有才呼叫 LLM
```

---

## 15. Rate Limit 設計

### 15.1 使用者層級

```text
免費版：每日 10 次 AI 分析
進階版：每日 100 次 AI 分析
專業版：每日 500 次 AI 分析
```

### 15.2 API 層級

```ts
@Throttle({ default: { limit: 10, ttl: 60_000 } })
@Post('/ai-advice/stocks/:symbol')
generateStockAdvice() {}
```

---

## 16. 錯誤處理

### 16.1 LLM Timeout

回傳：

```json
{
  "success": false,
  "error": {
    "code": "LLM_TIMEOUT",
    "message": "AI 分析暫時逾時，請稍後再試。"
  }
}
```

### 16.2 JSON Parse Error

處理方式：

1. 重試一次。
2. 若仍失敗，改用 fallback template。
3. 寫入 error log。

### 16.3 資料不足

LLM 應輸出：

```json
{
  "summary": "目前資料不足，不建議做出高信心操作判斷。",
  "action_level": "wait",
  "confidence": 0.3
}
```

---

## 17. 前端顯示策略

### 17.1 不要只顯示買賣

錯誤做法：

```text
買進
賣出
```

正確做法：

```text
狀態：分批停利
建議：可考慮賣出 1/3
理由：短線過熱，但趨勢尚未轉壞
回補條件：回測 20 日線不破
風險：若爆量跌破 5 日線需小心
```

### 17.2 UI 顯示區塊

```text
AI 結論卡片
操作建議卡片
技術理由
籌碼理由
市場理由
新聞理由
風險提醒
回補計畫
免責聲明
```

---

## 18. Push Notification 文案

### 18.1 漲多提醒

```text
良維短線漲幅偏大，已進入分批停利區。
可考慮先賣出 1/3，剩餘部位續抱。
```

### 18.2 回補提醒

```text
國巨已回測接近 20 日線，若尾盤站穩，可觀察買回 1/3。
```

### 18.3 跌破防守提醒

```text
健鼎跌破 20 日線且量能放大，趨勢有轉弱風險，建議降低部位。
```

### 18.4 大盤風險提醒

```text
台股今日大漲後短線偏熱，今日重點不是追高，而是檢查持股是否需要分批停利。
```

---

## 19. LLM 測試案例

### 19.1 測試案例一：漲多

Input：

```json
{
  "profit_pct": 15,
  "rsi14": 82,
  "bias20": 14,
  "signal_type": "TAKE_PROFIT"
}
```

期望：

```text
建議分批停利，不得建議全部賣出，不得說一定會跌。
```

### 19.2 測試案例二：回補

Input：

```json
{
  "pullback_from_high_pct": 10,
  "near_ma20": true,
  "rsi14": 52,
  "signal_type": "BUYBACK_ZONE"
}
```

期望：

```text
建議觀察回補 1/3，但必須附上失效條件。
```

### 19.3 測試案例三：資料不足

Input：

```json
{
  "quote": null,
  "technical": null,
  "news": []
}
```

期望：

```text
回答資料不足，建議等待，不得編造資料。
```

---

## 20. 建議實作順序

### Step 1

建立 `AiAdviceModule`

```text
ai-advice.module.ts
ai-advice.controller.ts
ai-advice.service.ts
llm-provider.interface.ts
openai.provider.ts
prompt-builder.ts
context-builder.ts
schemas/
```

### Step 2

先做單股 AI 分析

```http
POST /ai-advice/stocks/:symbol
```

### Step 3

加入 JSON Schema 驗證

- 使用 Zod 驗證輸出
- 不合格就 retry
- 仍失敗就 fallback

### Step 4

儲存 AI 建議

寫入：

```text
ai_advices
```

### Step 5

加入 Daily Advice

```http
POST /ai-advice/daily
```

### Step 6

加入交易日誌分析

```http
GET /trade-journals/analysis
```

### Step 7

加入 RAG

- 新聞向量化
- 交易日誌向量化
- 策略知識庫向量化

---

## 21. 最小可行 Prompt 範例

這是 MVP 可以先用的版本。

```text
你是台股波段操作輔助 AI。

請根據資料產生繁體中文建議。
你不能保證漲跌，不能叫使用者歐印，必須使用分批操作與風險控管語氣。

資料：
{{context}}

請輸出：
1. 總結
2. 狀態：等待／續抱／分批停利／觀察回補／降低風險
3. 建議動作
4. 三個理由
5. 風險提醒
6. 如果賣出，之後什麼條件可以買回
```

---

## 22. 結論

本 App 的 LLM 串接應採用：

```text
規則引擎先判斷
LLM 再解釋
JSON Schema 控制格式
Safety Guard 控制風險語氣
Prompt Version 控制可維護性
RAG 後期再加入
```

這樣的架構最適合做成作品集，因為它同時展現：

- AI 應用能力
- 後端架構能力
- 金融資料處理能力
- Prompt Engineering
- 風險控管設計
- React Native App 實作能力
