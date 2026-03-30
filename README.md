# 🤖 DayTrade Bot — Algorithmic Trading System (B3)

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python" />
  <img src="https://img.shields.io/badge/FastAPI-0.100+-green?style=flat-square&logo=fastapi" />
  <img src="https://img.shields.io/badge/Docker-ready-blue?style=flat-square&logo=docker" />
  <img src="https://img.shields.io/badge/Status-Live%20🟢-brightgreen?style=flat-square" />
  <img src="https://img.shields.io/badge/Mercado-B3%20WIN%2FWDO-orange?style=flat-square" />
</p>

> Sistema de day trade algorítmico operando ao vivo no mercado futuro brasileiro (WIN — Índice Bovespa e WDO — Dólar Futuro) com capital real.

---

## 🎯 O que é este projeto?

Bot de trading 100% automatizado que:

- Monitora o mercado em tempo real a cada 10 minutos
- Analisa múltiplas estratégias simultaneamente
- Toma decisões de compra/venda com base em regras quantitativas
- Gerencia o risco automaticamente (stop, limite de perda, tamanho de posição)
- Expõe um dashboard web com P&L, logs e métricas ao vivo
- Roda em cloud com auto-restart e zero intervenção manual

---

## 🏗️ Arquitetura

```
┌─────────────────────────────────────────────────┐
│                   FastAPI Backend                │
│  ┌─────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Engines │  │   Risk   │  │  Market Data  │  │
│  │ (7 strat│  │ Manager  │  │  (multi-src)  │  │
│  └────┬────┘  └────┬─────┘  └──────┬────────┘  │
│       └────────────┴───────────────┘            │
│                     │                           │
│              ┌──────▼──────┐                    │
│              │  Scheduler  │  (ciclo 10min)     │
│              └──────┬──────┘                    │
│                     │                           │
│          ┌──────────▼──────────┐                │
│          │   State / Database  │  (PostgreSQL)  │
│          └─────────────────────┘                │
└─────────────────────────────────────────────────┘
          │                        │
   Dashboard Web              Brokers API
   (HTML/JS/CSS)         (Alpaca · Binance · BTG)
```

---

## 📈 Estratégias Implementadas

### 1. ORB — Opening Range Breakout (WIN)
Captura rompimentos da faixa de preço dos primeiros 30 minutos de pregão.
- Filtro de regime via ADX (evita operar em mercado sem tendência)
- Filtro direcional via KST D1 (permite só LONG ou só SHORT conforme tendência semanal)
- Janela de entrada: 09:30–12:00 BRT
- Stop baseado na faixa do ORB

### 2. VWAP Reversion (WDO)
Opera reversões ao VWAP quando o preço se desvia significativamente.
- Filtro de slope direcional (bloqueia entradas contra a tendência de curto prazo)
- Desvio mínimo configurável (evita ruído)
- Filtro KST D1 sincronizado com ORB

### 3. Momentum (Crypto)
Captura movimentos direcionais em ativos cripto.
- Classificação de regime: Bull / Bear / Sideways
- Filtro EMA20 diária (bloqueia operações contra a tendência macro)
- Multi-ativo: BTC, ETH, BNB, SOL, XRP

### 4. Liquidity Sweep
Identifica varredura de stops em extremos de range e opera a reversão.

### 5. Mean Reversion
Reversão à média com Bollinger Bands — funciona bem em regimes laterais.

### 6. Regime Detection
Classifica o regime atual de mercado e ajusta o comportamento das demais estratégias.

### 7. Squeeze (Bollinger + Keltner)
Detecta compressão de volatilidade e opera o rompimento após o squeeze.

---

## 🛡️ Gerenciamento de Risco

| Parâmetro | Descrição |
|---|---|
| Stop por trade | Configurável por estratégia |
| Limite de perda diária | Encerra operações ao atingir X% do capital |
| Redução semanal | Reduz tamanho de posição se semana negativa |
| Tamanho dinâmico | Escala posição conforme crescimento do capital |
| Max drawdown | Suspende trading se drawdown > threshold |

---

## 🔧 Stack Técnico

| Camada | Tecnologia |
|---|---|
| Backend | Python 3.11 + FastAPI |
| Banco de dados | PostgreSQL + JSON state |
| Dados de mercado | yfinance, Alpha Vantage, Binance API |
| Execução | Alpaca API, BTG (simulado), Binance |
| Frontend | HTML + Vanilla JS + CSS |
| Deploy | Docker + Railway |
| Agendamento | APScheduler (ciclo 10min) |

---

## 📊 Performance (ao vivo desde Mar/2026)

```
Capital inicial: R$ 500,00
Capital atual:   R$ 436,20
Trades ao vivo:  5 (2 WIN, 3 WDO)
Win Rate:        20% (amostra pequena — aguardando 30-50 trades)
```

> ⚠️ Performance passada não garante resultados futuros. Este é um projeto educacional/experimental.

---

## 🚀 Como Rodar

```bash
# 1. Clone o repositório
git clone https://github.com/[SEU-USERNAME]/daytrade-bot.git
cd daytrade-bot

# 2. Crie o ambiente virtual
python -m venv venv
.\venv\Scripts\Activate.ps1   # Windows
source venv/bin/activate       # Linux/Mac

# 3. Instale as dependências
pip install -r requirements.txt

# 4. Configure as variáveis de ambiente
cp .env.example .env
# edite .env com suas chaves de API

# 5. Inicie o bot
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

---

## 📁 Estrutura do Projeto

```
daytrade_bot/
├── app/
│   ├── main.py              # FastAPI app + scheduler principal
│   ├── engines/             # Estratégias de trading
│   │   ├── orb30_win.py     # ORB para WIN
│   │   ├── vwap_reversion.py
│   │   ├── momentum.py
│   │   ├── regime.py
│   │   └── ...
│   ├── brokers/             # Integrações com brokers
│   ├── core/                # Config, database
│   └── market_data.py       # Coleta de dados
├── dashboard-web/           # Frontend do dashboard
├── data/                    # State files e logs
├── tests/                   # Testes automatizados
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

---

## 📄 Licença

Projeto privado — uso educacional e pessoal.

---

<p align="center">Desenvolvido por <a href="https://github.com/[SEU-USERNAME]">[SEU NOME]</a></p>
