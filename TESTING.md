# Testing Sequence & Validation Checklist

## Phase 1: Data Collection Testing (Days 1-2)

### 1.1 Price Feed Validation

**Test:** Manual trigger of main workflow

**Expected Airtable Updates:**
- `Gas_Tracker`: New entry every 5 minutes
- `Real_Time_Monitor`: Price quotes from all exchanges

**Validation Steps:**

1. **Check Gas_Tracker table:**
   - [ ] Verify `gas_price_gwei` is realistic (5-150 gwei range)
   - [ ] Confirm `gas_cost_per_trade` calculation (should be $5-50 typically)
   
2. **Check Real_Time_Monitor for each exchange:**
   - [ ] Uniswap prices present
   - [ ] SushiSwap prices present  
   - [ ] 1inch quotes present
   - [ ] OKX DEX quotes present
   - [ ] Kraken prices present
   
**Debug if missing:**
- Check API credentials in each node
- Verify rate limits aren't exceeded
- Test individual HTTP nodes manually

### 1.2 Price Normalization Test

**Expected behavior:**
- All prices should be within 5% of each other for same token
- Outliers should be filtered

**Validation Query:**
```sql
SELECT pair, buy_price, sell_price, buy_exchange, sell_exchange 
FROM Real_Time_Monitor 
WHERE timestamp > [last hour]
ORDER BY pair, buy_price
```

**Red flags:**
- [ ] Prices varying by >10% between exchanges (likely bad data)
- [ ] Null or 0 prices
- [ ] Same price across all exchanges (likely stale)

---

## Phase 2: Arbitrage Detection (Day 3)

### 2.1 Opportunity Identification

**Test:** Run with `TEST_MODE=true` in "Test Calculate Arbitrage" node

**Expected in Real_Time_Monitor:**
- [ ] `profit_potential` > 0 for some records
- [ ] `execution_status` = 'Skipped' or 'Executed'
- [ ] `skip_reason` populated when skipped

**Validation Query:**
```sql
SELECT COUNT(*), execution_status, 
       AVG(profit_potential), AVG(net_profit_estimate)
FROM Real_Time_Monitor
WHERE timestamp > [last 24h]
GROUP BY execution_status
```

**Success Criteria:**
- [ ] At least 20% opportunities with `profit_potential` > 0.1
- [ ] Some opportunities marked as 'Executed' (paper trades)

### 2.2 Gas Optimization Testing

**Trigger:** Gas_Optimization workflow

**Expected in Gas_Optimization table:**
- [ ] `recommended_threshold` adjusting based on gas prices
- [ ] `execution_window` changing throughout day
- [ ] `missed_opportunities_count` > 0 during high gas

**Validation:**
- [ ] Low gas (<20 gwei): threshold should be 0.2-0.4%
- [ ] High gas (>50 gwei): threshold should be >1.0%

---

## Phase 3: ML Training (Days 4-5)

### 3.1 Initial ML Threshold Training

**Requirement:** At least 10 records in Real_Time_Monitor

**Test:** Run ML_Training workflow manually

**Expected in ML_Thresholds:**
- [ ] `optimal_threshold` between 0.2 and 2.0
- [ ] `confidence` > 0 but < 1
- [ ] `f1_score` between 0 and 1
- [ ] `training_size` >= 10

**Debug checks:**
- If threshold = 0.5 exactly, likely using fallback
- If confidence = 0.1, insufficient data

### 3.2 ML Performance Validation

**After 50+ trades:**
```sql
SELECT 
  COUNT(*) as total,
  SUM(CASE WHEN net_profit_estimate > 0 THEN 1 ELSE 0 END) as profitable,
  AVG(dynamic_threshold) as avg_threshold
FROM Real_Time_Monitor
WHERE execution_status = 'Executed'
  AND timestamp > [last 24h]
```

**Success metrics:**
- [ ] Profitable rate > 60%
- [ ] Average threshold adapting to conditions

---

## Phase 4: Risk Management (Day 6)

### 4.1 Risk Calculation Testing

**Trigger:** Risk_Management workflow

**Expected in Risk_Management table:**
- [ ] `risk_status` = 'Safe' initially
- [ ] `max_position_size` = 5000 (10% of 50k portfolio)
- [ ] `daily_loss_limit` = 1000 (2% of portfolio)

**Test scenarios:**
1. [ ] Simulate losses by manually adding negative trades to Executed_Trades
2. [ ] Verify risk_status changes to 'Warning' at 50% of loss limit
3. [ ] Verify 'Critical' at 80% of loss limit
4. [ ] Verify 'Halted' at 100% of loss limit

### 4.2 Position Limits Testing

**Check Position_Limits table:**
- [ ] Shows exposure per trading pair
- [ ] `utilization_percent` calculates correctly
- [ ] `risk_tier` adjusts (Conservative/Moderate/Aggressive)

---

## Phase 5: Exchange Performance (Day 7)

### 5.1 Exchange Ranking Validation

**Trigger:** Exchange_Performance workflow

**Validation in Exchange_Performance table:**
- [ ] `execution_success_rate` between 0-100
- [ ] `avg_spread_percentage` realistic (0.05-2.0%)
- [ ] `reliability_rating` matches success rate

**Quality checks:**
- [ ] Excellent: success_rate > 70%
- [ ] Good: success_rate > 50%  
- [ ] Fair: success_rate > 20%
- [ ] Poor: success_rate < 20%

---

## Phase 6: Integration Testing (Days 8-9)

### 6.1 End-to-End Paper Trading

**Test sequence:**
1. Let main workflow run for 2 hours
2. Check complete data flow

**Opportunities table should show:**
- [ ] New opportunities every 5 minutes
- [ ] Mix of 'Executed' and 'Skipped' statuses
- [ ] Realistic profit/loss distribution

**Validation Query:**
```sql
SELECT 
  DATE(timestamp) as day,
  COUNT(*) as opportunities,
  SUM(CASE WHEN is_profitable THEN 1 ELSE 0 END) as profitable,
  SUM(net_profit_estimate) as total_profit
FROM Opportunities
GROUP BY DATE(timestamp)
```

### 6.2 Dashboard Validation

**Access:** `http://[your-n8n-url]/webhook/dashboard`

**Check all sections load:**
- [ ] 24H metrics calculating correctly
- [ ] Chart displaying 7-day history
- [ ] ML confidence showing
- [ ] Exchange rankings present
- [ ] Recent activity updating

**Common issues:**
- Null values breaking calculations
- Date parsing errors
- Missing data causing blank sections

---

## Phase 7: Performance Benchmarking (Day 10)

### 7.1 System Metrics

**Track over 24 hours:**
- [ ] Opportunities detected per hour
- [ ] Execution rate
- [ ] Average spread captured
- [ ] Gas costs vs profits

**Expected ranges:**
- [ ] 10-50 opportunities/hour (depends on volatility)
- [ ] 20-40% execution rate
- [ ] 0.2-0.5% average spread after costs
- [ ] Gas should be <30% of gross profit

### 7.2 Profitability Analysis

**Calculate actual vs theoretical:**
```sql
SELECT 
  SUM(gross_profit) as theoretical,
  SUM(net_profit_estimate) as after_gas,
  SUM(realized_profit) as actual_paper,
  COUNT(*) as total_trades,
  AVG(slippage_cost) as avg_slippage
FROM Real_Time_Monitor
WHERE execution_status = 'Executed'
  AND timestamp > [last 7 days]
```

**Red flags:**
- [ ] Actual < 50% of theoretical (slippage too high)
- [ ] Most trades < $5 profit (gas eating profits)
- [ ] Win rate < 50% (model needs retraining)

---

## Debug Checklist for Common Issues

### No opportunities detected
- [ ] Verify `TEST_MODE=true` in Test Calculate Arbitrage node
- [ ] Check price feeds are returning data
- [ ] Verify spread calculation in Calculate Arbitrage node
- [ ] Lower thresholds temporarily

### ML not training
- [ ] Need minimum 10 records in Real_Time_Monitor
- [ ] Check data types (`profit_potential` must be numeric)
- [ ] Verify timestamp formats are consistent

### Gas optimization not working
- [ ] Check Etherscan API key is valid
- [ ] Verify Gas_Tracker has recent entries
- [ ] Check calculation uses correct gas limit (200000)

### Risk management too aggressive
- [ ] Verify portfolio value is set correctly
- [ ] Check daily P&L calculation timeframe
- [ ] Ensure Position_Limits updating properly

### Dashboard not loading
- [ ] Check webhook URL configuration
- [ ] Verify all 7 data sources returning data
- [ ] Check browser console for JavaScript errors
- [ ] Verify Chart.js CDN is accessible

---

## Testing Notes

**Record keeping:**
- Document each test result with timestamp
- Screenshot any errors for debugging
- Save successful validation queries
- Note any parameter adjustments made

**Success criteria for production:**
- All phases complete without critical errors
- Consistent profitability in paper trading
- Risk management properly limiting exposure
- ML model showing improvement over static thresholds
