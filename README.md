An end-to-end data collection, cleaning, and analysis project examining whether U.S. weather
conditions measurably influence agricultural commodity futures prices.

**Hypothesis:** adverse weather (drought, storms, excessive rainfall) disrupts crop yields and
creates supply uncertainty, pushing futures prices up; favorable growing weather raises expected
supply and pressures prices down. The notebooks test this against USDA price data, NOAA storm
event records, and FRED corn price history.

---

## Repository layout

```
notebooks/
  data_collection.ipynb   Data collection + cleaning, inconsistency write-up
  analysis.ipynb          Full submission: parsers + 5 insights + 3 visualizations
data/
  raw/
    usda_price_forecasts.csv     USDA ERS season-average price forecasts (~16 MB)
    fred_corn_price.csv          FRED PMAIZMTUSDM monthly corn price index, 1990–present
  processed/
    usda_price_forecasts_clean.csv   Output of data_parser()
    fred_corn_price_derived.csv      Output of extra_source1() — returns, log-diffs, rolling stats
    noaa_storm_events.csv            Output of web_parser1() — storm events, Cook County IL
    weatherstack_corn_belt.csv       Output of web_parser2() — weather, 9 corn-belt cities
```

## Data sources

| # | Source | Method | Notes |
|---|--------|--------|-------|
| 1 | [USDA ERS — Season-Average Price Forecasts](https://www.ers.usda.gov/data-products/season-average-price-forecasts) | Downloaded CSV | Corn, Cotton, Soybeans, Wheat: basis, WASDE forecasts, prices received |
| 2 | [NOAA NCDC Storm Events](https://www.ncdc.noaa.gov/stormevents/) | HTML scrape (BeautifulSoup) | Cook County, Illinois; Feb 2024 – Feb 2025 |
| 3 | [Weatherstack API](https://weatherstack.com/documentation) | REST API | Current conditions for 9 top corn-producing metros |
| 4 | [FRED — PMAIZMTUSDM](https://fred.stlouisfed.org/series/PMAIZMTUSDM) | Downloaded CSV | Global corn price index, monthly |

## Pipeline

- **`data_parser(file_name)`**: USDA data. Fills `futures_exchange` per commodity (`CBOT` for
  wheat classes, `None` where a futures market doesn't apply), drops the >90%-null
  `commodity_class` column, coerces `marketing_year` from float to clean string (strips trailing
  `.0`), and backfills `unit` with the standard 5,000-bushel contract size.

- **`web_parser1(link)`**: Scrapes the NOAA storm events table, extracting place, event type,
  date, and magnitude. Reorders `MM/DD/YYYY` into a datetime index and substitutes `"None"` for
  blank magnitude strings.
- **`web_parser2(locations)`**: Queries Weatherstack for temperature, humidity, wind speed, and
  UV index across the corn belt, rate-limited to one request per second.

- **`extra_source1(link)`**: FRED corn prices. Derives percent change, log return, log-difference
  (for stationarity), and 30-period rolling mean/std.

## Analysis 

**Insights**

1. Worst-performing marketing year by futures basis: 2010 across all four commodities; cotton
   shows the lowest dispersion.
2. WASDE forecast accuracy: percent error between forecast and price actually received; forecasts
   skew toward overestimation.
3. Basis volatility by marketing year (`mean`/`std`/`min`/`max`, filtered to std > 0.5): 2007,
   2010, and 2012 are the most volatile, all with negative mean basis.
4. Correlation between monthly storm-event counts and 12-month rolling corn return volatility
   (~0.5).
5. Storm event distribution by type and year: convective events (thunderstorm wind, hail, flood)
   account for >75% of incidents during the growing season.
