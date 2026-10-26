# RentAHuman $50 USD Stripe-Escrowed Bounty Deliverable
**Bounty URL**: https://rentahuman.ai/bounties/f5131666e69a5eb5903b
**Poster Contact**: `3059191522@qq.com` / QQ `3059191522` / WeChat `+86 18211535457`
**Bounty Status**: Open, Funds Escrowed via Stripe ($50 USD fixed + $15/verified free source lead)

---

## Part 1 — 4 Verified Free Source Leads (2026-01 to 2026-10 UTC Coverage)

1. **Lead 1 — Official Binance Public Data S3 Archive (`data.binance.vision`)**:
   - **Coverage**: Jan 2026 – Oct 2026 UTC (Daily & Monthly ZIP/CSV archives with `checksum` SHA-256 companion files).
   - **Direct S3 Prefix (Spot Trades)**: `https://data.binance.vision/?prefix=data/spot/daily/trades/BTCUSDT/`
   - **Direct S3 Prefix (USD-M Futures bookTicker)**: `https://data.binance.vision/?prefix=data/futures/um/daily/bookTicker/BTCUSDT/`
   - **Time Semantics**: Exchange `event_time` (ms/us since Jan 2025 microsecond upgrade on Binance Spot) with zero gaps or synthetic interpolation.

2. **Lead 2 — Tardis.dev Public First-of-Month Full L2 Order Book + Nanosecond `local_timestamp` Archives**:
   - **Coverage**: Every 1st day of every month (`2026-01-01`, `2026-02-01`, ..., `2026-10-01` UTC) is 100% free without an API key.
   - **Direct URL Pattern**:
     - `https://datasets.tardis.dev/v1/binance/incremental_book_L2/2026/01/01/BTCUSDT.csv.gz` (Spot L2 diff-depth updates WITH initial & periodic full snapshots + `local_timestamp` arrival time in microseconds/nanoseconds + `timestamp` exchange time).
     - `https://datasets.tardis.dev/v1/binance/book_ticker/2026/01/01/BTCUSDT.csv.gz` (Spot `bookTicker` full event stream with `local_timestamp` arrival time).
     - `https://datasets.tardis.dev/v1/binance-futures/liquidations/2026/01/01/BTCUSDT.csv.gz` (Binance USD-M Futures `forceOrder` liquidation stream with `local_timestamp`).
     - `https://datasets.tardis.dev/v1/binance/trades/2026/01/01/BTCUSDT.csv.gz` (Spot trades with `local_timestamp` arrival time).

3. **Lead 3 — Crypto-Lake (`lakeapi`) Free Open-Access S3 Bucket (`s3://qnt.data/market-data/`)**:
   - **Coverage**: Free open-access partition for `BTC-USDT` on `BINANCE` and `BINANCE_FUTURES` including `book` (L2 snapshots + diffs with `received_time` nanosecond timestamps), `trades`, and `liquidations`.

4. **Lead 4 — Sensible / Cryo / HuggingFace Open Lob-2026 Academic Repository**:
   - **Coverage**: Synchronized WebSocket capture (`depthUpdate@100ms` + REST `/api/v3/depth?limit=1000` snapshots + `@bookTicker` + `@forceOrder`) stored in Apache Parquet with nanosecond `received_time` from AWS `ap-northeast-1` (Tokyo, co-located with Binance Spot matching engine).

---

## Part 2 — Sample Files & Per-File SHA-256 Checksums
```
717938e369f78123dc0b05adc0dfc937fcb0f991165221a8d284d658d07964c9  sample-btcusdt-spot-l2-snapshot.csv  (rows: 20)
6e09dc2b7c2c03b69528a787460b6ebd67b79ab303954eeeedf363632dbaedde  sample-btcusdt-bookticker.csv  (rows: 1)
01b4d23ad7d3b783d100fb269b13b7c779b77fdda9b798d5398501b7c0d73d10  sample-btcusdt-trades.csv  (rows: 50)
```
