## External Drivers

| Driver Family                           | Recommended Source        | Free / Paid | Authentication              | Data Format          | Update Frequency   | Notes                                                              |
| --------------------------------------- | ------------------------- | ----------- | --------------------------- | -------------------- | ------------------ | ------------------------------------------------------------------ |
| **Crude Oil Benchmark**                 | EIA API                   | ✅ Free      | API Key (free registration) | JSON                 | Daily              | Official US Energy Information Administration. Brent & WTI prices. |
| **Feedstock Cost Signal**               | Internal Borouge Source   | 🏢 Internal | Internal credentials        | CSV / Database / API | Depends on Borouge | Likely comes from SAP, Data Lake, or licensed provider.            |
| **Market Benchmark Signal**             | ICIS                      | 💰 Paid     | Enterprise API Key          | JSON / CSV           | Daily              | Industry-standard petrochemical benchmark prices.                  |
| **Country GDP / Market Intelligence**   | World Bank Indicators API | ✅ Free      | None                        | JSON / XML           | Quarterly / Annual | Good macroeconomic indicator.                                      |
| **Geopolitical / Macro-political News** | GDELT                     | ✅ Free      | None                        | JSON / CSV           | Every 15 minutes   | Global news event database with sentiment and themes.              |
| **FX Rates**                            | IMF Data API (IFS/SDMX)   | ✅ Free      | None                        | JSON / XML           | Daily              | Official exchange rate data.                                       |
| **Weather / Seasonal**                  | Open-Meteo                | ✅ Free      | None                        | JSON                 | Hourly / Daily     | No API key required for most usage.                                |
| **Country Holiday Calendar**            | Python `holidays` library | ✅ Free      | None                        | Python Objects       | Yearly             | Offline library, no API calls needed.                              |
## Crude Oil Imports API (EIA)

### Purpose
Retrieves monthly crude oil import statistics from the U.S. Energy Information Administration (EIA). This API is commonly used as an external signal for demand forecasting, supply chain analysis, and commodity price trend analysis.

---
### API Contract

| Field           | Value                                            |
| --------------- | ------------------------------------------------ |
| Provider        | U.S. Energy Information Administration (EIA)     |
| Method          | GET                                              |
| Endpoint        | `https://api.eia.gov/v2/crude-oil-imports/data/` |
| Authentication  | API Key (`api_key` query parameter)              |
| Response Format | JSON                                             |
| Data Frequency  | Monthly                                          |

---

### Input Parameters

| Parameter            | Type    | Required | Description                     |
| -------------------- | ------- | -------- | ------------------------------- |
| `api_key`            | String  | Yes      | EIA API Key                     |
| `frequency`          | String  | Yes      | Data frequency (`monthly`)      |
| `data[]`             | Array   | Yes      | Metric to retrieve (`quantity`) |
| `start`              | String  | Yes      | Start month (`YYYY-MM`)         |
| `end`                | String  | Yes      | End month (`YYYY-MM`)           |
| `sort[0][column]`    | String  | No       | Sort column                     |
| `sort[0][direction]` | String  | No       | Sort direction (`asc`/`desc`)   |
| `offset`             | Integer | No       | Pagination offset               |
| `length`             | Integer | No       | Number of records to fetch      |

---

### cURL

```bash
curl --location 'https://api.eia.gov/v2/crude-oil-imports/data/?api_key=<API_KEY>&frequency=monthly&data[0]=quantity&start=2021-01&end=2026-04&sort[0][column]=period&sort[0][direction]=desc&offset=0&length=5000' \
--header 'Accept: application/json'
```

---

### Output Format

```json
{
  "response": {
    "total": 5000,
    "dateFormat": "YYYY-MM",
    "frequency": "monthly",
    "data": [
      {
        "period": "2026-04",
        "country": "Canada",
        "destination": "PADD 2",
        "type": "Crude Oil",
        "grade": "Heavy",
        "quantity": 145678
      }
    ]
  }
}
```

---

### Key Output Fields

| Field | Description |
|-------|-------------|
| `period` | Reporting month |
| `country` | Country of crude oil origin |
| `destination` | Import destination/region |
| `type` | Crude oil type |
| `grade` | Crude oil grade |
| `quantity` | Imported crude oil quantity |

---

### Primary Use Cases

- Demand forecasting
- Oil supply monitoring
- Commodity price forecasting
- External market signal generation
- Energy market analytics

## Feedstock Cost Signal
It will be coming from some internal documentation so manual or some other thing will have to be done for this thing
## Geopolitical / macro-political news- GDELT News API (DOC 2.0)

### Purpose
Retrieves global news articles and news trend signals from the Global Database of Events, Language, and Tone (GDELT). Commonly used as an external signal for demand forecasting, geopolitical risk analysis, supply chain monitoring, and macroeconomic trend analysis. 

---
### API Contract

| Field | Value |
|-------|-------|
| Provider | GDELT Project |
| Method | GET |
| Endpoint | `https://api.gdeltproject.org/api/v2/doc/doc` |
| Authentication | None |
| Response Format | JSON |
| Data Frequency | Near Real-Time (rolling ~3 months of news) |

---

### Input Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| `query` | String | Yes | Search query (e.g., `India economy`) |
| `mode` | String | Yes | Output mode (`ArtList`, `TimelineVol`, `TimelineTone`, etc.) |
| `format` | String | Yes | Response format (`json`) |
| `startdatetime` | String | No | Start timestamp (`YYYYMMDDHHMMSS`) |
| `enddatetime` | String | No | End timestamp (`YYYYMMDDHHMMSS`) |
| `timespan` | String | No | Relative time window (e.g., `7d`, `1month`) |

---

### cURL

```bash
curl --location 'https://api.gdeltproject.org/api/v2/doc/doc?query=India%20economy&mode=TimelineTone&format=json' \
--header 'Accept: application/json'
```

---

### Output Format

```json
{
  "timeline": [
    {
      "date": "2026-06-01",
      "value": -0.42
    },
    {
      "date": "2026-06-02",
      "value": 0.18
    }
  ]
}
```

> **Note:** The response schema varies depending on the selected `mode`. For example, `ArtList` returns articles, while `TimelineTone` returns daily sentiment values and `TimelineVol` returns daily news volume.

---

### Key Output Fields

| Field | Description |
|-------|-------------|
| `date` | Reporting date |
| `value` | Daily tone/sentiment or news volume (depends on mode) |
| `title` | Article headline (`ArtList` mode) |
| `url` | News article URL (`ArtList` mode) |
| `domain` | News source domain |
| `language` | Source language |
| `sourcecountry` | Country of the news publisher |

---

### Primary Use Cases

- Geopolitical risk monitoring
- Demand forecasting
- Supply chain disruption detection
- Macroeconomic trend analysis
- News sentiment analysis
- Event-driven forecasting

---
### Notes
- No API key is required.
- Public API is rate-limited to approximately **1 request every 5 seconds**.
- Supports multiple output modes including `ArtList`, `TimelineVol`, `TimelineTone`, and `ImageCollage`.
- The DOC 2.0 API primarily searches a rolling window of recent news coverage (approximately the last three months). For large-scale historical analysis, GDELT recommends using its bulk datasets or BigQuery exports. :contentReference[oaicite:2]{index=2}
## Country GDP / Market Intelligence
My feature set would be:
These would be the 5 most important factors for polymer industry
1. **Manufacturing PMI** _(Leading Indicator)_
2. **Manufacturing Value Added / Industrial Production**
3. **GDP Growth**
4. **Gross Fixed Capital Formation**
5. **Inflation**

#### If PMI is unavailable
Since official PMI data is usually **paid**, I would replace it with:
- Industrial Production Index (OECD/FRED)
- Manufacturing Value Added (World Bank)

| Indicator                                         | Why it is Important for Polymer Demand Forecasting                                                                                                                                                                                                                                                                         |                                                                            |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Manufacturing PMI (Leading Indicator)             | PMI measures manufacturing activity and is one of the earliest indicators of economic expansion or contraction. Since polymers are widely used in manufacturing industries such as automotive, packaging, and construction, changes in PMI often signal future changes in polymer demand before they appear in sales data. |                                                                            |
| Manufacturing Value Added / Industrial Production | This reflects the level of manufacturing output in a country. Higher manufacturing activity generally leads to increased consumption of raw materials, including polymers, making it a strong indicator of industrial demand.                                                                                              |                                                                            |
| GDP Growth                                        | GDP growth represents the overall health of a country's economy. A growing economy usually leads to higher industrial production, consumer spending, infrastructure investment, and consequently higher demand for polymer products.                                                                                       |                                                                            |
| Gross Fixed Capital Formation (Investment)        | This measures investments in infrastructure, factories, machinery, and construction projects. Increased capital investment typically drives demand for polymers used in pipes, cables, construction materials, automotive components, and industrial equipment.                                                            |                                                                            |
| Inflation (Consumer Price Index)                  | Inflation influences purchasing power and business costs. High inflation can reduce consumer spending and industrial activity, leading to lower demand for polymer-based products, while stable inflation generally supports consistent demand.                                                                            |                                                                            |

All of these is not available monthly we will have to do some feature engineering:

| Indicator                                         | Monthly Data Available?                                                                                                                                                                                                                                                                                                    | Source                                                                     |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Manufacturing PMI**                             | ✅ Yes                                                                                                                                                                                                                                                                                                                      | S&P Global (Paid), ISM (US)                                                |
| **Manufacturing Value Added**                     | ❌ No                                                                                                                                                                                                                                                                                                                       | World Bank (Annual)                                                        |
| **Industrial Production Index (IPI)**             | ✅ Yes                                                                                                                                                                                                                                                                                                                      | OECD, FRED, National Statistics                                            |
| **GDP Growth**                                    | ❌ Mostly Annual / Quarterly                                                                                                                                                                                                                                                                                                | World Bank (Annual), OECD/Trading Economics (Quarterly)                    |
| **Gross Fixed Capital Formation**                 | ❌ Annual / Quarterly                                                                                                                                                                                                                                                                                                       | World Bank, OECD                                                           |
| **Inflation (CPI)**                               | ✅ Yes                                                                                                                                                                                                                                                                                                                      | World Bank (annual), but monthly from IMF, FRED, OECD, national statistics |
```
import requests
import pandas as pd

# --------------------------------------------
# Configuration
# --------------------------------------------

COUNTRY = "IND"
BASE_URL = "https://api.worldbank.org/v2/country/{country}/indicator/{indicator}?format=json&per_page=200"

INDICATORS = {
    "gdp_growth": "NY.GDP.MKTP.KD.ZG",
    "manufacturing_value_added": "NV.IND.MANF.ZS",
    "industry_value_added": "NV.IND.TOTL.ZS",
    "gross_fixed_capital_formation": "NE.GDI.FTOT.ZS",
    "inflation": "FP.CPI.TOTL.ZG"
}


# --------------------------------------------
# Fetch World Bank Indicator
# --------------------------------------------

def fetch_indicator(country, indicator):

    url = BASE_URL.format(country=country, indicator=indicator)

    response = requests.get(url)
    response.raise_for_status()

    data = response.json()

    if len(data) < 2:
        return pd.DataFrame()

    rows = []

    for record in data[1]:

        if record["value"] is None:
            continue

        rows.append({
            "year": int(record["date"]),
            "value": record["value"]
        })

    df = pd.DataFrame(rows)

    return df.sort_values("year")


# --------------------------------------------
# Annual -> Monthly
# --------------------------------------------

def annual_to_monthly(df, feature_name):

    monthly = []

    for _, row in df.iterrows():

        for month in range(1, 13):

            monthly.append({

                "date": pd.Timestamp(
                    year=int(row["year"]),
                    month=month,
                    day=1
                ),

                feature_name: row["value"]

            })

    return pd.DataFrame(monthly)


# --------------------------------------------
# Main
# --------------------------------------------

monthly_features = None

for feature_name, indicator in INDICATORS.items():

    print(f"Downloading {feature_name}")

    annual_df = fetch_indicator(COUNTRY, indicator)

    monthly_df = annual_to_monthly(
        annual_df,
        feature_name
    )

    if monthly_features is None:

        monthly_features = monthly_df

    else:

        monthly_features = monthly_features.merge(
            monthly_df,
            on="date",
            how="outer"
        )


monthly_features = (
    monthly_features
    .sort_values("date")
    .reset_index(drop=True)
)

print(monthly_features.head(24))

# Save

monthly_features.to_csv(
    "india_macro_features_monthly.csv",
    index=False
)

print("\nSaved to india_macro_features_monthly.csv")
```
## FX rates
### We get 100 request/month free from this and we will not need these many so it is effectively free.


| Field                          | Value                                                                                                                                                                                                                  |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Purpose**                    | Retrieves historical and latest foreign exchange (FX) rates for one or more currencies against a base currency. Used to generate monthly exchange rate features for demand forecasting and import parity calculations. |
| **Provider**                   | ExchangeRate.host                                                                                                                                                                                                      |
| **Method**                     | `GET`                                                                                                                                                                                                                  |
| **Endpoint (Latest)**          | `https://api.exchangerate.host/live`                                                                                                                                                                                   |
| **Endpoint (Historical)**      | `https://api.exchangerate.host/{date}`                                                                                                                                                                                 |
| **Endpoint (Time Series)**     | `https://api.exchangerate.host/timeframe`                                                                                                                                                                              |
| **Authentication**             | API Key (`access_key`)                                                                                                                                                                                                 |
| **Frequency**                  | Daily                                                                                                                                                                                                                  |
| **Input Parameters**           | `access_key`, `source` (e.g., USD), `currencies` (comma-separated), `start_date`, `end_date`                                                                                                                           |
| **Response Format**            | JSON                                                                                                                                                                                                                   |
| **Output Fields**              | Date, Base Currency, Currency Pair, Exchange Rate                                                                                                                                                                      |
| **Typical Features Generated** | Monthly Mean, Month-End Rate, Monthly Min/Max, Standard Deviation (Volatility), Monthly % Change                                                                                                                       |
| **Business Use**               | Captures currency fluctuations affecting import parity, product pricing, customer purchasing power, and regional demand.                                                                                               |

---

### Sample cURL (Time Series)

```bash
curl --request GET \
"https://api.exchangerate.host/timeframe?access_key=YOUR_API_KEY&start_date=2026-06-01&end_date=2026-06-30&source=USD&currencies=INR,EUR,CNY,AED,BRL"
```

---

### Sample Request

| Parameter    | Example            |
| ------------ | ------------------ |
| `access_key` | `xxxxxxxxxxxxxxxx` |
| `source`     | `USD`              |
| `currencies` | `INR,EUR,CNY,AED`  |
| `start_date` | `2026-06-01`       |
| `end_date`   | `2026-06-30`       |

---

### Sample Response

```json
{
  "success": true,
  "timeseries": true,
  "start_date": "2026-06-01",
  "end_date": "2026-06-30",
  "source": "USD",
  "quotes": {
    "2026-06-01": {
      "USDINR": 86.12,
      "USDEUR": 0.87,
      "USDCNY": 7.18
    },
    "2026-06-02": {
      "USDINR": 86.18,
      "USDEUR": 0.88,
      "USDCNY": 7.16
    }
  }
}
```

---

### Integration Pattern
* **Ingestion:** Scheduled daily API ingestion (Python/Dataiku/API Connect)
* **Storage:** Data Lake or Feature Store
* **Transformation:** Aggregate daily rates into monthly features (mean, last, min, max, volatility, % change)
* **Consumption:** Used as external macroeconomic features in the polymer demand forecasting model.

This format is consistent with the API contracts you've already prepared for **EIA**, **World Bank**, and the other external signals in your documentation.


## Weather / seasonal
### Purpose

Retrieves **historical, current, and forecast weather data** from the Visual Crossing Weather API. It is commonly used as an external signal for demand forecasting, sales forecasting, logistics planning, inventory optimization, and weather-driven analytics.

---

### API Contract

| Field           | Value                                                                                                                  |
| --------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Provider        | Visual Crossing Weather                                                                                                |
| Method          | GET                                                                                                                    |
| Endpoint        | `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/{location}/{startDate}/{endDate}` |
| Authentication  | API Key (`key` query parameter)                                                                                        |
| Response Format | JSON (default), CSV also supported                                                                                     |
| Data Frequency  | Daily / Hourly / Current / Forecast                                                                                    |

---

### Input Parameters

| Parameter     | Type   | Required | Description                                                                                       |
| ------------- | ------ | -------- | ------------------------------------------------------------------------------------------------- |
| `location`    | String | Yes      | City, state, country, postal code, or latitude/longitude (e.g., `London,UK` or `12.9716,77.5946`) |
| `startDate`   | String | No       | Start date (`YYYY-MM-DD`)                                                                         |
| `endDate`     | String | No       | End date (`YYYY-MM-DD`)                                                                           |
| `key`         | String | Yes      | Visual Crossing API Key                                                                           |
| `unitGroup`   | String | No       | Units (`metric`, `us`, `uk`)                                                                      |
| `include`     | String | No       | Data to include (`days`, `hours`, `current`, `events`)                                            |
| `elements`    | String | No       | Comma-separated list of required weather fields                                                   |
| `contentType` | String | No       | Response format (`json` or `csv`)                                                                 |
| `lang`        | String | No       | Language for weather descriptions                                                                 |

---

### cURL

```bash
curl --location 'https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/London,UK/2025-01-01/2025-01-31?unitGroup=metric&include=days&key=<API_KEY>' \
--header 'Accept: application/json'
```

---

### Output Format

```json
{
  "queryCost": 1,
  "latitude": 51.5072,
  "longitude": -0.1276,
  "resolvedAddress": "London, England, United Kingdom",
  "days": [
    {
      "datetime": "2025-01-01",
      "tempmax": 8.5,
      "tempmin": 2.3,
      "temp": 5.6,
      "humidity": 81.2,
      "precip": 1.4,
      "precipprob": 65,
      "windspeed": 18.4,
      "cloudcover": 72.5,
      "conditions": "Partially cloudy"
    }
  ]
}
```

---

### Key Output Fields

| Field        | Description                      |
| ------------ | -------------------------------- |
| `datetime`   | Observation date                 |
| `temp`       | Average temperature              |
| `tempmax`    | Maximum temperature              |
| `tempmin`    | Minimum temperature              |
| `humidity`   | Relative humidity (%)            |
| `precip`     | Total precipitation              |
| `precipprob` | Probability of precipitation (%) |
| `snow`       | Snowfall amount                  |
| `windspeed`  | Average wind speed               |
| `winddir`    | Wind direction (degrees)         |
| `pressure`   | Atmospheric pressure             |
| `cloudcover` | Cloud cover (%)                  |
| `visibility` | Visibility                       |
| `uvindex`    | UV Index                         |
| `sunrise`    | Sunrise time                     |
| `sunset`     | Sunset time                      |
| `conditions` | Weather condition summary        |

---
### Primary Use Cases

* Demand forecasting
* Sales forecasting
* Retail inventory optimization
* Supply chain and logistics planning
* Weather impact analysis
* Energy demand forecasting
* Agricultural analytics
* Event planning and risk assessment

## Country specific Holiday Calendar

```
pip install holidays

----------------------------------
## FOR COUNTRY SPECIFIC:
----------------------------------

import holidays
india = holidays.country_holidays("IN", years=2026)
for date, holiday in india.items():
    print(date, holiday)

----------------------------------
## FOR ALL THE COUNTRIES TOGETHER:
----------------------------------
import holidays
import pandas as pd

rows = []

year = 2026

for country_code in holidays.list_supported_countries():
    try:
        h = holidays.country_holidays(country_code, years=year)

        for holiday_date, holiday_name in h.items():
            rows.append({
                "country_code": country_code,
                "holiday_date": holiday_date,
                "holiday_name": holiday_name
            })

    except Exception:
        pass

df = pd.DataFrame(rows)

print(df)

```

## Market- Benchmark signal
As per my knowledge ICIS data is present with borouge but there is an alternative for it if we want to use in future:
==World Bank Commodity Prices (Pink Sheet)==
### ICIS Market Benchmark Data

### Purpose
Retrieves market benchmark prices for commodities, chemicals, and energy products from ICIS. These benchmark prices are commonly used as external signals for demand forecasting, price elasticity analysis, supply chain planning, and market trend monitoring.

---

### API Contract

| Field | Value |
|-------|-------|
| Provider | ICIS |
| Method | GET / File Feed (depends on subscription) |
| Endpoint | Customer-specific (provided by ICIS) |
| Authentication | API Key / OAuth / Enterprise Credentials |
| Response Format | JSON / CSV / XML (depends on subscription) |
| Data Frequency | Daily / Weekly / Monthly (dataset dependent) |

---

### Input Parameters

| Parameter | Type | Required | Description |
|----------|------|----------|-------------|
| `commodity` | String | Yes | Commodity or benchmark identifier |
| `start_date` | Date | Yes | Start date |
| `end_date` | Date | Yes | End date |
| `region` | String | No | Geographic market |
| `currency` | String | No | Desired currency |

---

### cURL

```bash
# Customer-specific endpoint (example only)

curl --location 'https://<ICIS_ENDPOINT>/market-prices?commodity=<COMMODITY>&start_date=2026-06-01&end_date=2026-06-30' \
--header 'Authorization: Bearer <ACCESS_TOKEN>'
```

---

### Output Format

```json
{
  "commodity": "Polyethylene",
  "currency": "USD",
  "prices": [
    {
      "date": "2026-06-01",
      "benchmark_price": 1245.60
    }
  ]
}
```

---

### Key Output Fields

| Field | Description |
|-------|-------------|
| `commodity` | Commodity or benchmark name |
| `date` | Price date |
| `benchmark_price` | Benchmark market price |
| `currency` | Price currency |
| `region` | Market region |

---

### Primary Use Cases

- Demand forecasting
- Market benchmark monitoring
- Price elasticity analysis
- Commodity price forecasting
- Procurement planning
- Supply chain optimization


## Recommended Dataset Columns

### From EIA – Crude Oil Imports
| Column | Description |
|--------|-------------|
| `period` | Reporting month (YYYY-MM) — join key for monthly aggregation |
| `crude_country_origin` | Country of crude oil origin — useful for supply concentration/risk features |
| `crude_destination` | Import destination/PADD region |
| `crude_type` | Type of crude oil (e.g., Crude Oil) |
| `crude_grade` | Grade (Light/Heavy) — affects refining cost and feedstock economics |
| `crude_import_quantity` | Imported quantity — core demand/supply signal for feedstock cost forecasting |

### From Internal Borouge Source – Feedstock Cost
| Column | Description |
|--------|-------------|
| `feedstock_cost` | Internal feedstock price/cost signal (source TBD — SAP/Data Lake/licensed provider) |
| `feedstock_cost_date` | Date of the recorded cost, for monthly alignment with other features |

### From GDELT – Geopolitical/Macro News
| Column | Description |
|--------|-------------|
| `news_date` | Date of the news signal |
| `news_tone` | Daily sentiment/tone score (TimelineTone mode) — proxy for geopolitical risk sentiment |
| `news_volume` | Daily news volume (TimelineVol mode) — spikes can indicate disruption events |
| `news_sourcecountry` | Country of publisher — helps localize the signal to relevant markets |

### From World Bank – Macroeconomic Indicators (annual → interpolated monthly)
| Column | Description |
|--------|-------------|
| `gdp_growth` | Annual GDP growth rate — overall economic health proxy |
| `manufacturing_value_added` | % of GDP from manufacturing — direct proxy for industrial polymer demand |
| `industry_value_added` | % of GDP from industry overall (broader than manufacturing) |
| `gross_fixed_capital_formation` | Investment in infrastructure/construction — drives polymer use in pipes, cables, construction |
| `inflation_cpi` | Consumer Price Index — affects purchasing power and industrial spending |
| `manufacturing_pmi` *(if available/paid)* | Leading indicator of manufacturing activity — earliest signal of demand shifts |
| `industrial_production_index` *(PMI substitute)* | Monthly industrial output proxy when PMI is unavailable |

### From FX Rates (ExchangeRate.host)
| Column | Description |
|--------|-------------|
| `fx_monthly_mean` | Average exchange rate for the month — smooths daily volatility |
| `fx_month_end_rate` | Rate at month close — reflects most recent pricing conditions |
| `fx_monthly_min` / `fx_monthly_max` | Range of currency movement within the month |
| `fx_volatility` | Standard deviation of daily rates — currency risk indicator |
| `fx_pct_change` | Month-over-month % change — captures depreciation/appreciation trend |

### From Visual Crossing – Weather/Seasonal
| Column | Description |
|--------|-------------|
| `temp_avg` / `temp_max` / `temp_min` | Temperature stats — seasonal demand driver (e.g., construction, packaging) |
| `humidity` | Relative humidity — relevant for certain polymer applications/storage |
| `precip` / `precip_prob` | Precipitation amount/probability — affects logistics and construction activity |
| `windspeed` | Wind speed — can affect logistics/shipping schedules |
| `cloudcover` | Cloud cover % — secondary seasonal indicator |
| `conditions` | Text summary of weather conditions |

### From `holidays` Library – Country Holiday Calendar
| Column | Description |
|--------|-------------|
| `holiday_flag` | Binary flag (1/0) indicating if the date is a public holiday — affects working days per month |
| `holiday_name` | Name of the holiday, if applicable |
| `working_days_in_month` | Derived count of business days after removing holidays/weekends — normalizes monthly demand comparisons |

### From ICIS / World Bank Pink Sheet – Market Benchmark
| Column | Description |
|--------|-------------|
| `benchmark_commodity` | Commodity/product name (e.g., Polyethylene) |
| `benchmark_price` | Market benchmark price — primary price signal for elasticity modeling |
| `benchmark_currency` | Currency of the price |
| `benchmark_region` | Market region the price applies to |
