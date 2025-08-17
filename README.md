# Options Strategy Bot Trader

An automated options trading system with Interactive Brokers integration, built with FastAPI backend and React/TypeScript frontend.

## 🎯 Overview

This project implements an automated trading bot for options strategies, providing real-time market data analysis, strategy execution, and portfolio management through Interactive Brokers API.

## 🚀 Features

- **Automated Trading**: Execute complex options strategies automatically
- **Real-time Market Data**: Live market data streaming and analysis
- **Strategy Management**: Multiple pre-configured trading strategies
- **Risk Management**: Built-in risk controls and position monitoring
- **Portfolio Tracking**: Real-time P&L and position tracking
- **Web Interface**: Modern React-based dashboard for monitoring and control
- **Paper Trading**: Test strategies with simulated trading environment

## 🛠️ Tech Stack

### Backend
- **FastAPI**: High-performance Python web framework
- **ib_insync**: Interactive Brokers API integration
- **PostgreSQL**: Database for historical data and configuration
- **Redis**: Caching and real-time data management
- **Celery**: Distributed task queue for background processing

### Frontend
- **React**: Modern UI framework
- **TypeScript**: Type-safe JavaScript
- **Material-UI**: Component library for consistent design
- **Chart.js**: Data visualization
- **WebSocket**: Real-time data updates

## 📋 Prerequisites

- Python 3.9+
- Node.js 16+
- Interactive Brokers account with API access enabled
- TWS (Trader Workstation) or IB Gateway installed

## 🔧 Installation

### 1. Clone the repository
```bash
git clone https://github.com/Milofax/options-strategy-bot-trader.git
cd options-strategy-bot-trader
```

### 2. Setup Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Setup Frontend
```bash
cd frontend
npm install
```

### 4. Configure Environment Variables
Create `.env` files in both backend and frontend directories:

**backend/.env**
```env
DATABASE_URL=postgresql://user:password@localhost/options_bot
REDIS_URL=redis://localhost:6379
IB_HOST=127.0.0.1
IB_PORT=7497  # Paper trading port (7496 for live)
IB_CLIENT_ID=1
SECRET_KEY=your-secret-key
```

**frontend/.env**
```env
REACT_APP_API_URL=http://localhost:8000
REACT_APP_WS_URL=ws://localhost:8000/ws
```

## 🚦 Getting Started

### 1. Start Interactive Brokers Gateway/TWS
Ensure IB Gateway or TWS is running and API connections are enabled.

### 2. Initialize Database
```bash
cd backend
python scripts/init_db.py
```

### 3. Start Backend Server
```bash
cd backend
uvicorn main:app --reload --port 8000
```

### 4. Start Frontend Development Server
```bash
cd frontend
npm start
```

### 5. Access the Application
Open your browser and navigate to `http://localhost:3000`

## 📊 Trading Strategies

The bot supports various options strategies including:
- Covered Calls
- Cash-Secured Puts
- Iron Condors
- Credit Spreads
- Straddles/Strangles
- Custom strategies via configuration

## ⚙️ Configuration

Trading strategies and risk parameters can be configured in `backend/config/strategies.yaml`:

```yaml
strategies:
  - name: "Iron Condor SPY"
    symbol: "SPY"
    type: "iron_condor"
    parameters:
      dte_min: 30
      dte_max: 45
      delta_short: 0.20
      delta_long: 0.10
      max_loss: 500
      profit_target: 0.50
```

## 🔒 Security & Risk Management

- Position size limits
- Maximum daily loss limits
- Strategy-specific risk parameters
- Automatic stop-loss orders
- Real-time margin monitoring
- Paper trading mode for testing

## 📝 Documentation

For detailed documentation, see the `/docs` directory:
- [Project Requirements](docs/prd.md)
- [UI/UX Specification](docs/ui-ux-specification.md)
- [API Documentation](docs/api.md)

## 🧪 Testing

### Backend Tests
```bash
cd backend
pytest tests/
```

### Frontend Tests
```bash
cd frontend
npm test
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## ⚠️ Disclaimer

**IMPORTANT**: This software is for educational purposes only. Trading options involves substantial risk of loss and is not suitable for all investors. Past performance is not indicative of future results. Always do your own research and consider consulting with a financial advisor before making investment decisions.

The developers of this software are not responsible for any financial losses incurred through the use of this bot. Use at your own risk.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Interactive Brokers for their comprehensive API
- The ib_insync community for excellent Python integration
- All contributors and testers

## 📧 Contact

For questions or support, please open an issue on GitHub.

---
**Note**: This bot requires an active Interactive Brokers account with API access permissions. Ensure you understand the risks involved in automated trading before using this system with real money.