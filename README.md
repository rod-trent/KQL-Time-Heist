# KQL Time Heist: Using the MarketLogs Dataset

Welcome, time bandits! This `Readme.md` guides you through using the `MarketLogs_1929.csv` dataset to pull off your KQL time heist, as outlined in the blog post *KQL Time Heist: Stealing Insights from History*. With this dataset, you’ll query historical stock market data from October 1929 to steal insights and rewrite history—without tripping over a null value!

## Dataset Overview

The `MarketLogs_1929.csv` dataset represents stock market activity in New York during October 1929, a pivotal month leading to the infamous Wall Street Crash. It’s your *Data Vault*, packed with trade records and investor gossip ripe for KQL queries.

### Schema
The dataset has the following columns:

| Column Name      | Type     | Description                              |
|------------------|----------|------------------------------------------|
| `Timestamp`      | datetime | When the trade occurred (e.g., `1929-10-01T09:00:00`) |
| `StockSymbol`    | string   | Stock ticker (e.g., `FORD`, `GE`, `RCA`, `USST`) |
| `Price`          | real     | Stock price in USD (e.g., `22.50`)       |
| `TradeVolume`    | int      | Number of shares traded (e.g., `15000`)  |
| `InvestorNote`   | string   | Insider tips or gossip (e.g., `Strong demand for Model A`; can be empty) |

### Sample Data
```
Timestamp,StockSymbol,Price,TradeVolume,InvestorNote
1929-10-01T09:00:00,FORD,22.50,15000,"Strong demand for Model A"
1929-10-01T09:15:00,GE,395.25,8000,"Radio boom driving growth"
1929-10-01T09:30:00,RCA,101.75,12000,"Buy RCA before radio peaks"
```

## Prerequisites

To use this dataset with KQL, you’ll need:
- **Access to a Kusto-compatible environment**: Azure Data Explorer, Azure Monitor Logs, or a local Kusto emulator.
- **KQL knowledge**: Familiarity with basic KQL operators like `where`, `summarize`, `search`, and `top`. Check the [KQL documentation](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/) for a refresher.
- **The dataset**: Download `MarketLogs_1929.csv` from this repository.

## Setup Instructions

Follow these steps to load and query the dataset:

1. **Download the Dataset**
   - Grab `MarketLogs_1929.csv` from this repository or the blog post’s resources.
   - Ensure the file is accessible in your working environment.

2. **Ingest the Dataset into Kusto**
   - **Option 1: Azure Data Explorer**
     - Log into your Azure Data Explorer cluster.
     - Create a table to hold the data:
       ```kql
       .create table MarketLogs (
           Timestamp: datetime,
           StockSymbol: string,
           Price: real,
           TradeVolume: int,
           InvestorNote: string
       )
       ```
     - Ingest the CSV file using the ingestion command:
       ```kql
       .ingest inline into table MarketLogs <|
       @'path/to/MarketLogs_1929.csv'
       ```
       Replace `path/to/MarketLogs_1929.csv` with the file’s location (e.g., a blob storage URL or local path if supported).
   - **Option 2: Local Kusto Emulator**
     - If using a local emulator, import the CSV via the emulator’s ingestion tools or convert it to a format like JSON for manual loading.
     - Example for manual loading (simplified):
       ```kql
       .create table MarketLogs (...)
       .ingest inline into table MarketLogs <|
       1929-10-01T09:00:00,FORD,22.50,15000,"Strong demand for Model A"
       ...
       ```

3. **Verify the Data**
   - Run a quick query to ensure the data loaded correctly:
     ```kql
     MarketLogs
     | take 5
     ```
   - This should display the first five rows, matching the sample data above.

## Using the Dataset

Now that your *Data Vault* is ready, use KQL to raid it! The blog post provides three challenges to get you started, but here’s how to approach querying the dataset generally:

### Example Queries
1. **Find the Top Traded Stocks**
   - Goal: Identify the most traded stocks on October 1, 1929.
   - Query:
     ```kql
     MarketLogs
     | where Timestamp between (datetime(1929-10-01)..datetime(1929-10-02))
     | summarize TotalVolume = sum(TradeVolume) by StockSymbol
     | top 3 by TotalVolume desc
     ```
   - Expected Output: Stocks like `FORD` or `RCA` with high trade volumes.

2. **Search for Crash Rumors**
   - Goal: Find investor notes mentioning “crash” in October 1929.
   - Query:
     ```kql
     MarketLogs
     | where Timestamp between (datetime(1929-10-01)..datetime(1929-10-31))
     | search "crash" in (InvestorNote)
     | project Timestamp, StockSymbol, InvestorNote
     ```
   - Expected Output: Notes like “Heard crash rumors, sell FORD.”

3. **Spot the Priciest Stock**
   - Goal: Find the stock with the highest average price in early October.
   - Query:
     ```kql
     MarketLogs
     | where Timestamp between (datetime(1929-10-01)..datetime(1929-10-10))
     | summarize AvgPrice = avg(Price) by StockSymbol
     | order by AvgPrice desc
     | take 1
     ```
   - Expected Output: Likely `GE` with a high average price.

### Tips for Querying
- **Handle Nulls**: The `InvestorNote` column may be empty. Use `isnotnull(InvestorNote)` to filter non-empty notes if needed.
- **Date Ranges**: Use `between` for precise time filtering, as shown above.
- **Experiment**: Try operators like `join`, `count`, or `bin` to uncover new insights, such as daily price trends or trade spikes.

## Heist Log: Share Your Insights

After running your queries, share your loot in the blog post’s *Heist Log*! Post your best KQL query, the insight you stole, and any hilarious blunders (e.g., “I typed `summarise` instead of `summarize` and got stuck in a 1920s speakeasy!”). Format your entry like this:

- **Query Used**: (Your KQL query)
- **Insight Stolen**: (What you discovered)
- **Blunder Bonus**: (Any query fails or funny moments)

## Troubleshooting

- **Ingestion Errors**: Ensure the CSV’s column headers match the table schema. Check for stray commas or quotes in the data.
- **Query Fails**: Double-check syntax (e.g., `summarize`, not `summarise`). Use `| take 10` to inspect data for unexpected formats.
- **No Results**: Verify the date range in `where` clauses. The dataset covers October 1–31, 1929, only.
- **Need Help?**: Refer to the [Azure Data Explorer docs](https://docs.microsoft.com/en-us/azure/data-explorer/) or ping the blog post’s comment section.

## Ready to Heist?

With `MarketLogs_1929.csv` loaded and KQL at your fingertips, you’re set to raid the Data Vault of 1929. Steal those insights, dodge the timeline cops, and share your loot in the Heist Log. Let’s rewrite history—one query at a time!

*“Query fast, or you’re history!”*
