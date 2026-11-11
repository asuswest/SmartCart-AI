# 🛒 SmartCart AI — Local Grocery Planner (v0.1)

A **working local-first proof of concept** for your Windows 11 PC, accessible from
an Android phone on your **trusted home Wi-Fi**. Built with Python, Streamlit and
SQLite. The local grocery locations are starter entries for the Lakeville, MN area.

> **IMPORTANT:** The application does **not** automatically search all grocery
> sites, extract restricted flyer data, or claim access to live store prices.
> You enter prices, import verified CSVs, or configure a **permitted** HTTPS CSV
> price feed. Sample prices are fictional and cannot be used unless you
> explicitly enable demo mode in the planner.

## What works now

- Track pantry products and the unit each uses (gal, dozen, lb, oz, each, etc.).
- Estimate remaining stock by subtracting expected consumption each day.
- Correct the estimate by conducting a stock count. When counts are at least two
  days apart, recorded purchases help automatically refine average daily usage.
- Replenish when projected inventory will not last to the next shopping trip,
  plus a safety buffer. Adjust the stock-up target.
- Add verified weekly prices manually or import them using CSV.
- Display price per unit as well as total whole-package cost; support package
  sizes, required brands, local offer dates, loyalty-account
  eligibility, price freshness, and demo-source warnings.
- Exactly enumerate combinations of up to 5 selected stores (from at most 12
  price-bearing stores); pick the lowest-cost **whole packages** for each item.
- Add a configurable penalty for each **extra stop**.
- Optionally calculate the shortest round trip from user-entered **directed
  travel times**, then include your value of travel time in the decision score.
- Show a comparison of feasible one-, two-, and three-stop plans (and up to
  five stops if requested), itemized
  store lists, and a downloadable CSV shopping list.
- Record items actually purchased and add the package quantities to inventory.
- Optionally update prices from **authorized direct CSV feeds** (not scrapers).

## Step 1 — Install Python on Windows 11

Install **Python 3.11 or newer** from https://www.python.org/downloads/windows/
Check *Add python.exe to PATH* if offered. Then open PowerShell in the extracted
`smartcart_ai` project folder (in File Explorer, click the address bar, type
`powershell`, and press Enter).

Check installation:

```powershell
py -3 --version
```

**Easiest option:** Double-click **`setup_windows.bat`** in the extracted project
folder. It performs Step 2 and runs the included tests for you. Or continue
with the manual commands below if you prefer full control.

## Step 2 — Create a virtual environment

All commands in this README are commands **you type during setup**, not actions
an AI agent can execute without your approval.

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Dependencies download once during installation. Running the local program does
not need an internet connection, except to obtain real prices from permitted
online feeds, or for your own browsing of retailer ads.

## Step 3 — Start the local PC dashboard

Double-click **start_local.bat**, or run:

```powershell
.\.venv\Scripts\python.exe -m streamlit run app.py --server.address 127.0.0.1 --server.port 8501
```

Open: **http://127.0.0.1:8501**

The first run creates `data/smartcart.sqlite3` and populates **store locations
only**. You must add your own pantry products and actual prices. For a quick
safe experiment, open **Overview → Load example pantry and demo prices**, then
check **Show fictional demo prices** under **Shopping planner**.

## Step 4 — Make it available on your Android phone

This is for your **trusted private home network**, not hotel/public Wi-Fi.
Your PC must be on and connected to the same network as your phone.

1. In PowerShell, create a one-session password (replace with a long unique one):

   ```powershell
   $env:GROCERY_APP_PASSWORD = "REPLACE-WITH-A-LONG-UNIQUE-PASSWORD"
   .\start_lan.bat
   ```

2. On the PC, run `ipconfig` and find the **IPv4 Address** for your Wi-Fi or
   Ethernet adapter, e.g. `192.168.1.123` (this is just an example).
3. On Android, join the **same trusted Wi-Fi** and open
   `http://<YOUR-PC-IP>:8501` in Chrome, e.g. `http://192.168.1.123:8501`.
4. Sign in with the household password. In Chrome, use the menu → **Add to Home
   screen** if you want an app-like shortcut.
5. If Windows Firewall blocks access, allow TCP 8501 for **Private networks,
   Local Subnet only**, and **never** for Public networks. An Administrator
   PowerShell command is:

   ```powershell
   New-NetFirewallRule -DisplayName "SmartCart AI (Private LAN)" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8501 -Profile Private -RemoteAddress LocalSubnet
   ```

**Security:** The passphrase stops casual access, but simple LAN HTTP is **not
end-to-end encrypted**. Only use it on a network you trust. Never forward port
8501 on your router, disable Windows firewall, turn off Streamlit CORS/XSRF,
or expose the application directly to the internet. If your PC is off or your
phone is on cellular instead of local Wi-Fi, the app will not be accessible.
For secure access away from home, a private VPN plus HTTPS/authentication is
an **additional project**, not configured in this starter version.

## Step 5 — Add and calibrate your groceries

Open **My pantry**. Use one physical unit per product throughout the program:

| Product | Tracking unit | Daily usage illustration |
|---|---|---:|
| Milk | gal | 0.20 |
| Eggs | dozen | 0.12 |
| Chicken | lb | 0.35 |
| Coffee | oz | 1.00 |

These are **fictional household rates**, not real estimates of your habits.
Start with your own amounts; if unsure, set the rate to 0. Later, periodically
record **actual stock counts**. The program uses the count and logged purchases
since the previous verified count to adjust usage by exponential smoothing.
The rate cannot be learned reliably from purchases alone; users may stockpile
or waste items. The app refuses to learn a negative rate caused by unrecorded
purchases or inconsistent counts.

Once a product is projected to have less than `shopping interval + buffer`
days of stock at shopping time, it suggests buying enough to reach the target
stock level. Quantities are editable before planning.

## Step 6 — Enter real prices from flyers

Start with the local grocery retailers you actually use. Open their official
weekly ads and product sites in your browser, verify the **selected local store**,
and record a price. You may create an offer one by one in **Prices & flyers**, or
export your own observations into a CSV matching `samples/offers_template.csv`.

**CSV columns**:

```csv
store_id,product_name,brand,pack_qty,unit,price,valid_from,valid_to,loyalty_required,source,source_url
```

**Example ONLY; fictional price** (when actually importing such a row it is treated
as real user-supplied data, so do NOT use made-up prices for real plans):

```csv
aldi-lakeville,Milk,Generic,1,gal,3.29,2026-11-11,2026-11-17,false,Manually checked weekly ad,https://www.aldi.us/
```

**Important constraints:**

- Add the product to the pantry first; the CSV product name must match.
- Use the same unit as the pantry (milk `gal` cannot be imported as `fl oz`
  unless **you** convert the package size correctly).
- `pack_qty` is quantity **in that pantry unit**, `price` is the cost for
  the **entire package**, and `source` identifies where you saw the price.
- Brand match is exact when you set a required brand in the pantry.
- Set `loyalty_required=true` only if the deal requires a card/coupon. In **Stores
  & travel**, check whether you have that store's loyalty account.
- Use ISO dates `YYYY-MM-DD`; unknown date fields may be blank. Expired prices
  aren't used; prices too old (default 14 days) aren't used.
- Prices can still be out of stock or store-specific. Check at checkout.
- If you enter a revised price for an identical item/pack from the **same
  source**, the program replaces the older quote, preventing contradictory
  prices from that source. A repeated identical import is ignored and doesn't
  artificially reset its freshness.

You can add further stores under **Stores & travel → Add another local grocery
store**. Use the new store ID in your imported CSVs and travel matrix.

**Grocery information sources worth checking manually**:

- Target: https://www.target.com/weekly-ad
- Cub: https://www.cub.com/
- Hy-Vee: https://www.hy-vee.com/
- ALDI: https://www.aldi.us/
- Walmart: https://www.walmart.com/

This application has **no guaranteed live API access** to these retailers.

## Step 7 — Optimize the shopping trip

Open **Shopping planner**:

1. Choose when you'll shop and the most stores you're willing to visit.
2. Set your **extra stop cost**, for example `$5` to represent the inconvenience
   of each additional shopping stop.
3. Check/edit the forecast suggestions or add a product manually.
4. Compare the feasible 1-, 2-, and 3-stop plans.
5. Download the store-by-store CSV. After shopping, check purchased products and
   record the restocks.

**Calculation**:

```
number_of_packs = ceiling(required_quantity / package_quantity)
line_cost       = number_of_packs × package_price
basket_cost     = sum(line_cost)
extra_stop_cost = (number_of_stores - 1) × your_stop_penalty
driving_value   = round_trip_minutes × your_value_per_hour / 60
score           = basket_cost + extra_stop_cost + driving_value
```

The optimizer chooses the **minimum score**, breaks ties in favor of fewer
stops, and always shows the **actual basket cost** separately from the score.
It considers only products with eligible offers. Any item without a verified,
valid offer is prominently listed as **unpriced** and is **NOT** silently assumed
free. Plans using unpriced items are therefore incomplete.

### Optional: drive times and visit order

Download `samples/travel_template.csv` from **Stores & travel** and supply directed
travel-time rows in minutes, such as:

```csv
from_id,to_id,minutes
HOME,aldi-lakeville,9
aldi-lakeville,HOME,9
HOME,cub-south,7
cub-south,HOME,7
aldi-lakeville,cub-south,6
cub-south,aldi-lakeville,6
```

These travel times are **illustrations, not measured routes**. Enter your own
actual values. The route search checks every store ordering and reports driving
time only when a **complete route** is available. For a 5-store plan, that may
require many pairs. If the complete directed matrix is unavailable, the planner
clearly labels routing unavailable; it does NOT invent a drive time.

## Step 8 — Optional weekly direct-CSV feed automation

You may connect structured data feeds when the provider gives you permission.
There is **no included unofficial scraper** and no built-in retailer login.

1. Copy `feeds.example.json` to `feeds.json` (not shared publicly).
2. Add one permitted feed to the `feeds` array, using its host and HTTPS URL:

   ```json
   {
     "feeds": [
       {
         "name": "My authorized weekly offers feed",
         "url": "https://example.com/approved-offers.csv",
         "allowed_host": "example.com",
         "licensed_or_permitted": true,
         "api_key_env": "MY_GROCERY_FEED_TOKEN"
       }
     ]
   }
   ```

3. Set the optional `MY_GROCERY_FEED_TOKEN` environment variable as required
   by your feed provider, and click **Refresh configured feeds** in **Prices &
   flyers**. If your feed doesn't need a bearer token, set `api_key_env` to `""`.
4. An authorized CSV feed must use the exact import schema and contain
   **`valid_to` on every row**. Dated expiry limits stale-feed problems.
5. To refresh with a weekly **Windows Task Scheduler** task, use your own
   scheduled command with the venv Python interpreter:

   ```powershell
   .\.venv\Scripts\python.exe refresh_feeds.py
   ```

   Schedule the command weekly via Task Scheduler, with **Start in** set to
   the extracted `smartcart_ai` folder. Only use this after configuring a
   permitted feed. The PC must be running for the scheduled job to execute.

**This feed adapter only knows how to import a specified HTTPS CSV.** Retailers'
APIs often return different schemas and use OAuth/client credentials. Each
source needs a documented, approved adapter and any necessary product matching
before being enabled. An API's mere existence is not permission for access.

## Tests

Run without the web interface:

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

Tests cover forecasting, learned usage, quantity rounding, missing/stale prices,
brand restrictions, stop penalties and route selection.

## Data location and backup

The SQLite file is `data/smartcart.sqlite3` (created automatically on first run).
The `data` directory, `.venv`, any `feeds.json`, and cache files are deliberately
excluded from source-control via `.gitignore`. Back up `data` regularly **when
the app is not running**, or make a SQLite backup with proper locking.

## Roadmap: what the next versions should add

1. **Reliable retailer connectors:** permissions review; feed adapters with
   store-specific pricing, expirations, stock checks and explicit data sources.
2. **Receipt capture:** upload a PDF/photo or import an itemized email receipt;
   match items for user approval before updating stock.
3. **Inventory intelligence:** handling of spoilage, irregular consumption,
   planned meals, holidays, guests, stockpiling, and confidence bounds.
4. **Smarter deals:** coupons, membership limits, `buy N get M`, sales taxes,
   per-pound weighted produce, exact SKU/UPC matching, and substitution rules.
5. **Automatic scheduling:** after authorized feeds are configured, generate
   a new week's shopping plan and save it locally with a dated snapshot.
6. **Optional local LLM via Ollama:** receipt categorization, natural-language
   questions, flyer text extraction with human validation. Not necessary for
   accurate basic optimization; never trust LLM-extracted prices unverified.
7. **Remote secure access:** private VPN and authenticated HTTPS (not open port
   forwarding), possibly an installable Progressive Web App in a later phase.

## Notes

This is an educational, single-household MVP. It has no warranty of store
pricing, stock availability, coupons, road conditions, or product quality. The
optimization is exact for **the simplified model provided**, not necessarily
for every real retail promotion. `SmartCart AI` is not affiliated with retailers.