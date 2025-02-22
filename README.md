# 📌 Dunum high frequency arbitrage bot

## 🚀 Project Overview

This project is a **high-frequency arbitrage bot** operating on the **Aptos blockchain**. The bot is designed to execute optimized transactions across multiple **DEXs (Decentralized Exchanges)** and **Binance** to capture price discrepancies and generate profits.

### ⚡ Key Features

- 📊 **Real-time price monitoring** across multiple platforms
- 🤖 **High-frequency arbitrage detection** (DEX → Binance & Binance → DEX)
- 🔄 **Automated transaction execution** with multi-threaded architecture
- 📈 **Dynamic volume & slippage management**
- 🔔 **Telegram notifications for executed arbitrages**
- 🔐 **Secure transactions with error monitoring**
- 🖥 **Runs an Aptos Full Node for optimal performance**

---

## 🛠️ Technical Architecture

### 📌 Technologies Used

- **Core Language**: `C++`
- **Transaction Automation**: `Node.js` & `TypeScript`
- **DEX API Integration**: `REST API` & `WebSockets`
- **Data Storage (Logs & PnL Tracking)**: `SQLite`
- **Real-time Notifications**: `Telegram Bot API`
- **Optimization**: `Multithreading` for ultra-fast execution

### 📌 System Diagram

```
+----------------------------------------------------+
| Binance WebSocket Listener                         |
| (Real-time price updates)                         |
+----------------------------------------------------+
          |                         |
          v                         v
+----------------------+  +-----------------------+
| DEX API Fetcher      |  | Arbitrage Detector    |
| (Cetus, Pancake, etc.) |  | (Price gap analysis) |
+----------------------+  +-----------------------+
          |                         |
          v                         v
+----------------------------------------------------+
| Trade Execution Engine                              |
| (Order management, volume calculations, gas fees) |
+----------------------------------------------------+
          |
          v
+----------------------------------------------------+
| Telegram Notifications & Log System                |
+----------------------------------------------------+
```

---

## ⏱️ Performance

- **Response Time**: ~8ms per arbitrage cycle
- **Success Rate**: ~98.4% on executed trades
- **Infrastructure**:
  - Runs an **Aptos Full Node** on the same server
  - Uses **8GB RAM & 400GB SSD** on a Contabo Cloud VPS
- **Threading Model**:
  - Each DEX runs on its **own thread** for maximum performance
  - **Mutexes** handle inter-thread communication
  - **Arbitrage detection** runs in a **dedicated thread**, scanning for opportunities **every millisecond**

- **First Month Results** 📈
  - With an initial capital of $1000, the bot generated a 30% return in its first month of operation.


---


## 🔍 Code Insights

### **1️⃣ Binance WebSocket Listener** (`binance_listener`)
```cpp
void binance_listener() {
    while (running) {
        double binance_price = get_binance_price("APT/USDC");
        aptusdc_binance.store(binance_price);
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}
```

### **2️⃣ DEX API Fetcher** (`dex_api_thread`)
```cpp
void dex_api_thread() {
    while (running) {
        for (const auto& dex : dex_list) {
            double buy_price, sell_price;
            std::tie(buy_price, sell_price) = fetch_dex_prices(dex);
            dex_prices[dex].buy_price.store(buy_price);
            dex_prices[dex].sell_price.store(sell_price);
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(5));
    }
}
```

### **3️⃣ Arbitrage Detection** (`watchdog.cpp`)
```cpp
void detect_arbitrage() {
    while (running) {
        for (const auto& dex : dex_list) {
            double profit = calculate_arbitrage_profit(dex);
            if (profit > MIN_PROFIT_THRESHOLD) {
                execute_trade(dex);
            }
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}
```

### **4️⃣ Trade Execution** (`execute_trade`)
```cpp
void execute_trade(std::string dex) {
    std::string txn_hash = submit_trade(dex);
    log_trade(txn_hash);
    send_telegram_notification("Arbitrage executed on " + dex);
}
```

---

## 📌 Usage Examples

### ✅ **Run in simulation mode**
```bash
./watchdog --simulate
```

### ✅ **Force trade on a specific DEX**
```bash
./watchdog --dex sushiswap --volume 5000
```

### ✅ **View transaction logs**
```bash
tail -f logs/arbitrage.log
```

---

## 🛡️ Security & Optimization

- **Smart slippage management** to prevent losses
- **Volume limits** based on available liquidity
- **Automatic error recovery for API failures**
- **Thread pool optimization** for fastest execution

---

## 📜 Info

📬 **Contact**: [maxime.lavail@hotmail.fr](mailto:maxime.lavail@hotmail.fr)

📌 **GitHub Repository**: [https://github.com/maximelavail/cpp.arbitrage_bot](https://github.com/maximelavail/cpp.arbitrage_bot)

