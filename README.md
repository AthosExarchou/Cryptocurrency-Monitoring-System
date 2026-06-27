# Real-Time Cryptocurrency Monitoring & Classification System

## Introduction
This project was developed as a group project for the **Services and Network Systems** course at [Harokopio University of Athens – Dept. of Informatics and Telematics](https://dit.hua.gr).
It demonstrates a real-time cryptocurrency monitoring, classification, and alerting system built with **[Node-RED](https://nodered.org/)**, **CoinGecko API**, **SQLite**, **K-Means clustering**, and **RabbitMQ**. The system includes a **dashboard** for real-time visualization and analytics.

The system:
- Fetches live cryptocurrency market data from the **CoinGecko API**.
- Stores historical values in an **SQLite database**.
- Applies **K-Means clustering** to detect fluctuation categories (small, medium, large).
- Provides real-time dashboards for visualization (table and charts).
- Sends alerts via **RabbitMQ**, differentiating between free and premium users.

---

## Data Source: CoinGecko API
The system retrieves data from the [CoinGecko API](https://www.coingecko.com/en/api/documentation), specifically:

```
https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=10&page=1&sparkline=false
```

Key fields used:
- **id** – unique identifier of the coin
- **name** – human-readable name
- **symbol** – ticker symbol
- **current_price** – live market price
- **market_cap** – market capitalization
- **total_volume** – 24h trading volume

---

## System Architecture & Flows

The system is built in **Node-RED** and consists of four main flows:

### 1. Flow 1: Data Ingestion & Classification
- Fetches live market data from CoinGecko.
- Parses JSON and formats into SQL insert statements.
- Stores each coin’s data with timestamp in SQLite.
- Data refresh every 5 minutes *(adjustable)*.

### 2. Flow 2: UI & Storage
- Provides the **Node-RED Dashboard** with:
  - Table showing current coin data.
  - Line chart showing historical price evolution.
- Allows manual queries of the database.

### 3. Flow 3: K-Means Clustering
- Groups historical prices **per coin** using K-Means.
- Defines fluctuation categories:
  - **Small** (low variance)
  - **Medium**
  - **Large** (high volatility)
- Updates clusters dynamically as new data arrives.

### 4. Flow 4: RabbitMQ Notification System
- Sends classification results as alerts via RabbitMQ.
- Implements **subscription differentiation**:
  - *Free users* subscribe to a single queue (e.g., `notifications.bitcoin`).
  - *Premium users* subscribe to all alerts via topic exchange (`notifications.*`).

---

## Database Schema

Historical data is stored in `coingecko_history`:

```sql
CREATE TABLE IF NOT EXISTS coingecko_history (
    entry_id INTEGER PRIMARY KEY AUTOINCREMENT,
    id TEXT,
    name TEXT,
    symbol TEXT,
    current_price REAL,
    market_cap REAL,
    volume_24h REAL,
    timestamp TEXT
);
```

This ensures per-coin **historical tracking**, enabling meaningful clustering.

---

## Clustering & Fluctuation Categories

The system applies **K-Means clustering** on each coin’s **historical prices**.  
Instead of static thresholds, categories are discovered dynamically:

- **Cluster centroids** represent different price ranges.
- Each new value is classified as belonging to the *nearest cluster*.
- Results are interpreted as *small*, *medium*, or *large* fluctuations.

This allows adaptive classification that evolves with the market.

---

## Repository Structure
```
.
├── flows.json             # Node-RED flow configuration
├── README.md              # Project documentation
├── report_el.pdf          # Final report in greek
└── (ignored) coingecko.db, alerts.log
```

> `coingecko.db` (SQLite database) and `alerts.log` are not included in the repo.  
> They are generated locally during execution.

---

## Notifications (RabbitMQ)

Classification results are broadcast to **RabbitMQ**.

- **Free Users**: Subscribed only to a specific coin queue (e.g., `alerts.bitcoin`).
- **Premium Users**: Subscribed to all alerts with wildcard topic `alerts.*`.

This enables differentiated alerting services.

---

## Visualization (Dashboard)

The Node-RED Dashboard provides:

1. **Table** – showing live cryptocurrency prices and their fluctuation categories.
2. **Line Chart** – plotting historical prices for each coin.
3. **Color-coded values** – easy identification of fluctuation categories.

Accessible at:
```
http://localhost:1880/ui
```

---

## How to Run

### Requirements
- Node-RED installed (`npm install -g node-red`)
- RabbitMQ installed locally or accessible remotely
- SQLite (lightweight, built-in)

### Steps
1. Place `flows.json` in your Node-RED user directory (e.g., `~/.node-red`).
2. Start Node-RED:
   ```bash
   node-red
   ```
3. Open the Node-RED editor at [http://localhost:1880](http://localhost:1880).
4. Access the Dashboard at [http://localhost:1880/ui](http://localhost:1880/ui).
5. Start RabbitMQ locally and access:
   - Management UI: [http://localhost:15672](http://localhost:15672)
   - HTTP API: [http://localhost:15672/api](http://localhost:15672/api)

> Note: If RabbitMQ runs remotely, replace `localhost` with the server IP.
> If you don’t use RabbitMQ, this step can be skipped.

---

## Outputs & Templates

The system generates multiple outputs:
- **SQLite DB (`coingecko_history`)** with all historical entries.
- **Dashboard visualizations** (Table + Chart).
- **Classification alerts** via RabbitMQ.
- **Log files** (e.g., `alerts.log`) for debugging.

---

## Areas for Improvement
- Expand clustering with more advanced ML models.
- Add anomaly detection alerts.
- Integrate user-defined coin watchlists.
- Deploy in Docker for reproducibility.

---

## Conclusion

This project demonstrates how **Node-RED**, **CoinGecko API**, **SQLite**, **K-Means**, and **RabbitMQ** can be combined into a robust **real-time monitoring and alerting system**.

Key strengths:
- Extensible architecture (new coins, new alerting rules).
- Adaptive clustering instead of static thresholds.
- Real-time dashboard and alerting integration.

It is a practical example of integrating **data engineering, real-time analytics, and messaging systems** for cryptocurrency monitoring.

---

## Author

- **Name**: Exarchou Athos
- **Student ID**: it2022134
- **Email**: it2022134@hua.gr or athosexarhou@gmail.com

## License
This project is licensed under the MIT License.
