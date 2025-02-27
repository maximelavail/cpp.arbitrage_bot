# 📌 Dunum High-Frequency Arbitrage Bot

## 🚀 Project Overview

This project is a **high-frequency arbitrage bot** operating on the **Aptos blockchain**. The bot is designed to execute optimized transactions across multiple **DEXs (Decentralized Exchanges)** and **Binance** to capture price discrepancies and generate profits efficiently.

⚠️ this readme is not the final documentation, it will be posted shortly, for the moment I wrote this quickly.

### Table of Contents
- 1. Technical Architecture
- 2. System Diagram
- 3. Code Insights
- 4. Algorithm for Transaction Volume Calculation
- 5. Performance Metrics
- 6. Security & Optimization
- 7. Contact & Repository

## 1 🛠️ Technical Architecture

- **Core Language**: `C++`
- **Transaction Automation**: `Node.js` & `TypeScript`
- **DEX API Integration**: `REST API` & `WebSockets` & `Local Aptos Full-Node`
- **Data Storage (Logs & PnL Tracking)**: `SQLite`
- **Real-time Notifications**: `Telegram Bot API`
- **Optimization**: `Multithreading` for ultra-fast execution

## 2 📌 System Diagram

```
+--------------------------------------------------------+
| Main Function                                          |
| (Thread Management)                                    |
+--------------------------------------------------------+
        |                        |
        v                        v
+------------------------+  +----------------------------+  
| DEX API Fetcher        |  | Binance WebSocket Listener |
| (Sushi, Pancake, etc.) |  | (Real-time price updates)  |
+------------------------+  +----------------------------+  
        |                        |
        v                        v
+--------------------------------------------------------+
| Arbitrage Detector (watchdog)                          |
| (Price gap analysis)                                   |
+--------------------------------------------------------+
                |
                v
+--------------------------------------------------------+
| Trade Execution Engine                                 |
| (Order management, volume calculations, gas fees)      |
+--------------------------------------------------------+
                |
                v
+--------------------------------------------------------+
| Telegram Notifications & Log System                    |
+--------------------------------------------------------+
```

## 3 🔍 Code Insights

### **1️⃣ Main Function** (`main`)
```cpp
int main(int argc, char* argv[]) {
    std::thread binance(binance_listener);
    std::thread dex_api(dex_api_thread);
    std::thread watchdog(watchdog_thread);
    std::thread reset_pnl(reset_pnl_daily_thread);
    std::thread rebalancing(rebalancing_thread);

    if (argc > 2 && std::string(argv[1]) == "--simulate") {
        simulate = true;
    }

    binance.join();
    dex_api.join();
    watchdog.join();
    reset_pnl.join();
    rebalancing.join();
}
```

### **2️⃣ Binance WebSocket Listener** (`binance_listener`)
```cpp
void binance_listener() {
	while (running) {
		for (const auto& pair : pair_list) {
			double dex_long_price, dex_short_price;
			std::tie(dex_long_price, dex_short_price) = get_binance_price("APT/USDC");

			binance[pair].dex_long_price.store(dex_long_price);
			binance[pair].dex_short_price.store(dex_short_price);
		}
	}
}
```

### **3️⃣ DEX API Fetcher** (`dex_api_thread`)
```cpp
void dex_api_thread() {
	while (running) {
		for (const auto& dex : dex_list) {
			double buy_price, sell_price;
			std::tie(buy_price, sell_price) = fetch_dex_prices(dex);

			dex_prices[dex].buy_price.store(buy_price);
			dex_prices[dex].sell_price.store(sell_price);
		}
	}
}
```

### **4️⃣  Arbitrage Detection** (`watchdog`)
```cpp
void watchdog() {
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


### **5️⃣ Trade Execution** (`execute_trade`)
```cpp
void execute_trade(std::string dex) {
	std::string txn_hash = submit_trade(dex);
	log_trade(txn_hash);
	send_telegram_notification("Arbitrage executed on " + dex);
}
```

## 4 🤖 Algorithm for Transaction Volume Calculation

### **Understanding Slippage in On-Chain Transactions**

The execution of an on-chain transaction is primarily governed by slippage tolerance. Slippage is determined by the liquidity reserves of the trading pool: the higher the pool reserves relative to the transaction volume, the greater the probability of successful execution.

The smart contract enforces a **maximum slippage threshold** to protect the integrity of the liquidity pool. If a transaction exceeds this threshold, it will fail.

#### **Formula for On-Chain Slippage Calculation:**
```
Slippage = 1 - ((Reserve_x * Reserve_y) / ((Reserve_x + Amount_in) * (Reserve_y - Amount_out)))
```

Using this formula, the smart contract determines whether a transaction can proceed.

### **Optimizing Volume Calculation**

To maximize the transaction acceptance rate (~98.4%), the bot dynamically adjusts the trade volume based on real-time pool reserves.

#### **Formula for Maximum Acceptable Trade Volume:**
```
Max_Volume = (Reserve_x * Slippage_Max) / (1 - Slippage_Max)
```

By applying this formula, the bot optimizes transaction sizes to ensure execution within the acceptable slippage limit. If the calculated volume exceeds available balance, the bot adjusts dynamically.

```cpp
if (max_swap > balance_usdc) {
    max_swap = std::min(balance_usdc, 1500000000.0);  // Cap max volume to avoid over-execution
}
```

## 5 ⏱️ Performance Metrics

- **Response Time**: ~8ms per arbitrage cycle
- **Success Rate**: ~98.4% on executed trades
- **Infrastructure**:
  - Runs an **Aptos Full Node** on the same server
  - Uses **8GB RAM & 400GB SSD** on a Contabo Cloud VPS
- **Threading Model**:
  - Each DEX runs on its **own thread** for maximum performance
  - **Mutexes** handle inter-thread communication
  - **Arbitrage detection** runs in a **dedicated thread**, scanning for opportunities **every millisecond**

### **First Month Results** 📈
  - With an initial capital of $1000, the bot generated a **30% return** in its first month of operation.

## 6 🛡️ Security & Optimization

- **Smart slippage management** to prevent losses
- **Volume limits** based on available liquidity
- **Automatic error recovery for API failures**
- **Thread pool optimization** for fastest execution

## 7 📜 Contact & Repository

📬 **Contact**: [maxime.lavail@hotmail.fr](mailto:maxime.lavail@hotmail.fr)

📌 **GitHub Repository**: [https://github.com/maximelavail/cpp.arbitrage_bot](https://github.com/maximelavail/cpp.arbitrage_bot)

