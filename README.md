# 🚀 Cross-DEX Arbitrage Bot - BETA

An advanced multi-exchange arbitrage bot built with N8N, featuring machine learning optimization, dynamic risk management, and comprehensive performance tracking.

## 📊 Overview

This bot monitors price discrepancies across multiple decentralized and centralized exchanges to identify and execute profitable arbitrage opportunities in real-time. Built with a modular architecture, it features ML-driven threshold optimization, gas-aware execution, and sophisticated risk management.

## 📚 Documentation

Full project documentation is maintained in Notion:

**[Velocity-BPA — Cross-DEX Arbitrage Bot Suite](https://www.notion.so/Velocity-BPA-Cross-DEX-Arbitrage-Bot-Suite-32b4035692b980f189cefe843421fd69)**

The Notion page covers system architecture, all eight n8n workflow descriptions, the Airtable data schema (9 tables), API integrations, the ML optimization engine, risk management framework, and key operational learnings.

## ✨ Features

### Core Capabilities
- **Multi-Exchange Integration**: Monitors Uniswap, SushiSwap, 1inch, OKX DEX, Kraken, and CoinGecko
- **Intelligent Price Aggregation**: Normalizes quotes across venues with outlier detection (IQR filtering)
- **Smart Execution Logic**: Paper trading mode with realistic slippage simulation
- **Cross-Chain Monitoring**: XRPL-Ethereum bridge opportunity detection

### Advanced Features
- **🤖 Machine Learning Optimization**
  - Dynamic threshold adjustment based on historical performance
  - F1 score optimization for precision/recall balance
  - Confidence-weighted decision making
  - Automatic retraining on 15-minute intervals

- **⛽ Gas Optimization Engine**
  - Real-time gas price monitoring via Etherscan
  - Execution window classification (Optimal/Good/Poor/Avoid)
  - Opportunity cost tracking for gas-related misses
  - Predictive optimal window forecasting

- **🛡️ Risk Management System**
  - Portfolio-based position sizing (10% max per position)
  - Daily loss limits (2% max drawdown)
  - Dynamic trade size adjustment based on risk status
  - Circuit breakers for critical market conditions

- **📈 Performance Analytics**
  - Exchange reliability rankings
  - Win rate tracking (overall and ML-guided)
  - Slippage analysis
  - 7-day performance trends

## 🏗️ Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Price Feeds    │────▶│  Normalization  │────▶│  Arbitrage Calc │
│  (6 exchanges)  │     │  & Filtering    │     │  & Execution    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                          │
                    ┌─────────────────┐                  ▼
                    │   ML Training    │◀────────┌─────────────────┐
                    │  (Threshold Opt) │         │    Airtable     │
                    └─────────────────┘         │   Data Store    │
                                                 └─────────────────┘
                                                          ▲
┌─────────────────┐     ┌─────────────────┐             │
│ Risk Management │────▶│ Gas Optimization│─────────────┘
└─────────────────┘     └─────────────────┘
```

## 📁 Workflow Components

| Workflow | Purpose | Schedule |
|----------|---------|----------|
| `main_arbitrage.json` | Core arbitrage detection and execution | Every 5 minutes |
| `ml_training.json` | ML threshold optimization | Every 15 minutes |
| `gas_optimization.json` | Gas price analysis and threshold adjustment | Every 15 minutes |
| `risk_management.json` | Portfolio risk assessment | Every 5 minutes |
| `exchange_performance.json` | Exchange reliability tracking | Every 4 hours |
| `dashboard.json` | Web-based monitoring dashboard | On-demand webhook |

## 🗂️ Data Schema

### Airtable Tables
- `Real_Time_Monitor` - All arbitrage opportunities
- `Executed_Trades` - Trade execution records
- `Gas_Tracker` - Historical gas prices
- `ML_Thresholds` - Model parameters
- `Risk_Management` - Portfolio risk metrics
- `Exchange_Performance` - Exchange reliability stats
- `Gas_Optimization` - Execution window analysis
- `Position_Limits` - Per-pair exposure tracking

## 🚦 Getting Started

### Prerequisites
- N8N instance (v1.0+)
- Airtable account with base configured
- API keys for:
  - Etherscan (gas prices)
  - 1inch (DEX aggregation)
  - OKX DEX (requires API credentials)
  - Optional: Kraken (for CEX prices)

### Installation

1. **Import Workflows**
   - Import all 6 workflow JSON files into N8N
   - Configure credentials for each service

2. **Setup Airtable**
   - Create a new base named "Arbitrage Bot"
   - Import the provided schema for all tables
   - Generate API key and configure in N8N

3. **Configure Parameters**
   ```javascript
   // In main_arbitrage workflow
   PORTFOLIO_SIZE = 50000  // Your portfolio size
   TEST_MODE = true        // Start with paper trading
   ```

4. **Activate Workflows**
   - Start with `main_arbitrage` workflow
   - Once data flowing, activate `ml_training`
   - Enable remaining workflows after validation

## 📊 Dashboard

Access the real-time dashboard at:
```
http://your-n8n-instance/webhook/dashboard
```

Features:
- 24-hour P&L tracking
- ML model performance metrics
- Exchange reliability rankings
- Gas optimization status
- Risk management indicators
- Recent trade activity feed

## 🧪 Testing Protocol

The bot includes a comprehensive 7-phase testing sequence:

1. **Data Collection** - Validate price feeds
2. **Arbitrage Detection** - Verify opportunity identification
3. **ML Training** - Confirm model optimization
4. **Risk Management** - Test safety mechanisms
5. **Exchange Performance** - Validate rankings
6. **Integration** - End-to-end testing
7. **Performance** - Benchmark profitability

See [TESTING.md](./TESTING.md) for detailed testing instructions.

## ⚠️ Risk Disclaimer

This bot is for educational purposes. Cryptocurrency arbitrage involves significant risks:
- Smart contract vulnerabilities
- MEV bot competition
- Impermanent loss
- Network congestion
- Exchange API failures

Always test thoroughly with small amounts before scaling up.

## 🔧 Configuration Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| `PORTFOLIO_SIZE` | 50000 | Total portfolio value in USD |
| `MAX_POSITION_PCT` | 0.10 | Max position size as % of portfolio |
| `DAILY_LOSS_LIMIT_PCT` | 0.02 | Max daily loss as % of portfolio |
| `GAS_LIMIT` | 200000 | Estimated gas per transaction |
| `MIN_PROFIT_USD` | 5 | Minimum profit to execute |
| `TEST_MODE` | true | Enable paper trading |

## 📈 Performance Metrics

Expected performance (based on paper trading):
- **Opportunities/Hour**: 10-50 (volatility dependent)
- **Execution Rate**: 20-40%
- **Average Spread**: 0.2-0.5% after costs
- **Win Rate**: 60-75% with ML optimization
- **Gas/Profit Ratio**: <30%

## 🤝 Contributing

Contributions welcome! Areas for improvement:
- WebSocket price feeds for lower latency
- Additional exchange integrations
- MEV protection strategies
- Cross-chain execution (not just monitoring)
- Advanced ML features (LSTM, ensemble methods)

## 📝 License

MIT License - see [LICENSE](./LICENSE) for details

## 🙏 Acknowledgments

- N8N community for workflow automation platform
- OpenZeppelin for smart contract best practices
- DeFi protocols for providing liquidity and arbitrage opportunities

## 📞 Support

For issues and questions:
- Open an issue in this repository
- Check [TESTING.md](./TESTING.md) for debugging guides
- Review Airtable logs for execution history

---

**⚡ Built with N8N | 🤖 Powered by Machine Learning | 📊 Tracked with Airtable**
