# 🏠 Airbnb Revenue \& Host Performance Analysis | Power BI

An end-to-end Power BI analytics project analyzing Airbnb revenue, host performance, and pricing strategy — built on a star-schema data model with 100,200 bookings across 80 properties, 50 hosts, and 20 NYC neighbourhoods. The project translates raw booking data into business insights, with each KPI validated against the source data before being reported.

\---

![Executive Overview](Page_1_-_Executive_Overview.png)
![Host & Property](Page_2_-_Host___Property.png)

\---

## 📌 Project Overview

|||
|-|-|
|**Tool**|Power BI Desktop|
|**Data Model**|Star schema — 1 fact table + 4 dimension tables|
|**Records**|100,200 bookings \| 80 properties \| 50 hosts \| 20 neighbourhoods|
|**Pages**|2 (Executive Overview, Host \& Property)|
|**Techniques Used**|DAX measures, decomposition tree, scatter analysis, custom price binning, custom Airbnb-branded theme (JSON)|

\---

## 🗂️ Data Model

```
fact\_bookings (100,200 rows)
   ├── dim\_date        (date, year, quarter, month, weekend/holiday flags)
   ├── dim\_host        (host info, response rate, superhost flag)
   ├── dim\_location     (neighbourhood, city, lat/long)
   └── dim\_property     (room type, property type, accommodates, base price)
```

**Key fact table fields:** `price`, `minimum\_nights`, `calculated\_revenue`, `review\_score`, `availability\_365`, `cancellation\_policy`, `booking\_source`

\---

## 📊 Dashboard Pages

### Page 1 — Executive Overview

High-level KPIs (Total Revenue, Avg Nightly Rate, Occupancy Rate, Avg Review Score, Cancellation Indicator), revenue trends by month/year, bookings by source and room type, and a RevPAR vs. Avg Nightly Rate dual-axis trend chart.

### Page 2 — Host \& Property

Property-type performance, Superhost vs. Regular host comparison, a price-band vs. review-score analysis, host performance leaderboard, and an interactive revenue decomposition tree (Neighbourhood → Room Type → Property Type → Superhost status).

\---

### KPI Card Formatting (Reference, YoY % and Conditional Formatting Measures)

Each KPI card on the Executive Overview page isn't just one measure — it's actually backed by a small group of supporting measures that handle the prior-year comparison, the variance arrow, and the red/green color coding you see under each number. I built this out as its own "KPI Formatting" folder in the measure table so it's reusable across cards instead of hardcoding values into each visual.

For every KPI (Avg Nightly Rate, Avg Review Score, Cancellation Rate, Occupancy Rate, Total Revenue), there's a consistent set of measures:

```dax
Avg Nightly Rate PY        = CALCULATE(\[Avg Nightly Rate], SAMEPERIODLASTYEAR(dim\_date\[full\_date]))

Avg Nightly Rate YoY %      = DIVIDE(\[Avg Nightly Rate] - \[Avg Nightly Rate PY], \[Avg Nightly Rate PY])

Avg Nightly Rate Ref Label  = "PY: $" \& FORMAT(\[Avg Nightly Rate PY], "#,##0") \&
                               " | Var: $" \& FORMAT(\[Avg Nightly Rate] - \[Avg Nightly Rate PY], "#,##0")
```

The same pattern repeats for **Avg Review Score**, **Cancellation Rate**, **Occupancy Rate**, and **Total Revenue** — just swapping the base measure each time. This is what's actually driving the small text under every KPI card (e.g. *"PY: $236 | Var: $-1"*).

On top of that, two more layers of measures control the visual styling itself:

```dax
CF Color - Avg Nightly Rate = 
IF(\[Avg Nightly Rate YoY %] >= 0, "#2E7D32", "#C62828")

CF Background - Avg Nightly Rate = 
IF(\[Avg Nightly Rate YoY %] >= 0, "#E8F5E9", "#FFEBEE")
```

These two get plugged into the card visual's **Conditional Formatting** (Format pane → Callout value / Background → Format by → Field value), so the card automatically turns green when a metric improved year-over-year and red when it dropped — without me touching the visual manually every time the data refreshes. Same logic is mirrored for every KPI on the page (Cancellation Rate, Occupancy Rate, Total Revenue, Avg Review Score), each with its own PY, YoY %, Ref Label, Color and Background measure.

\---

## 💡 Business Insights \& Recommendations

Beyond visualizing KPIs, this analysis was built to answer a few specific business questions: where is revenue actually concentrated, is the business leaving pricing power on the table, and can the dashboard's own metrics be trusted at face value? The five findings below come directly from the raw CSVs, cross-checked against the dashboard numbers.

### 1\. Just 10 hosts generate nearly half of all revenue

Out of **50 total hosts**, the **top 10 alone account for 47.9% of total revenue** ($84.3M of $176.08M).

**Insight:** Revenue is heavily concentrated in a small host base — a real single-point-of-failure risk if even a few of these hosts churn.
**Recommendation:** Build a dedicated retention program for top-tier hosts, and diversify host acquisition to reduce this dependency.

### 2\. Price has no relationship with guest satisfaction

Correlation between `price` and `review\_score` across all 100,200 bookings: **−0.002** — effectively zero.

**Insight:** Average review scores stay flat at \~3.9 across every price band, from Under $100 to $400+. Raising nightly rates carries no measurable risk of lower guest satisfaction.
**Recommendation:** This is a low-risk lever for revenue growth — underpriced hosts in particular can increase rates without harming their ratings.

### 3\. Entire Home/Apt drives disproportionate revenue

|Room Type|Revenue Share|Booking Share|Avg Revenue/Booking|
|-|-|-|-|
|Entire home/apt|77.7%|56.2%|**$2,430**|
|Private room|13.5%|31.2%|$758|
|Hotel room|7.2%|6.4%|$1,968|
|Shared room|1.7%|6.2%|$487|

**Insight:** Entire Home/Apt listings earn **3.2x more per booking** than Private Rooms, while requiring proportionally fewer bookings to generate the same revenue.
**Recommendation:** Prioritize host acquisition and marketing spend toward Entire Home/Apt inventory — it's the most revenue-efficient segment.

### 4\. The Superhost Paradox

||Revenue Share|Booking Share|Avg Revenue/Booking|
|-|-|-|-|
|Superhost|69.3%|71.1%|$1,713|
|Regular|30.7%|28.9%|**$1,865**|

**Insight:** Superhosts dominate booking volume, but Regular hosts actually earn **\~9% more per booking on average** — suggesting Superhosts may be competing more on visibility/price than per-transaction value.
**Recommendation:** Investigate whether Superhost listings are systematically underpriced relative to demand; there may be a monetization opportunity here.

### 5\. Revenue is concentrated in a handful of neighbourhoods

The top 5 neighbourhoods (Bronx, Chelsea, Crown Heights, Bushwick, Lower East Side) account for **\~47% of total revenue**, out of 20 neighbourhoods tracked.

**Insight:** Revenue skews heavily geographic; underperforming neighbourhoods may be under-marketed or carrying the wrong inventory mix.
**Recommendation:** Study the top neighbourhoods' inventory mix (room type, pricing) and test replicating it in lower-performing areas.

\---

## 🛠️ Tools \& Skills Demonstrated

* Star-schema data modeling (fact/dimension design)
* DAX (AVERAGEX, SUMX, DIVIDE, SWITCH, SAMEPERIODLASTYEAR, calculated columns, custom binning)
* Conditional formatting via DAX (dynamic color/background measures for KPI cards)
* Data validation — confirming each KPI's definition matches what the underlying data can actually support
* Custom JSON theme application (Airbnb brand styling)
* Decomposition tree, dual-axis trend charts, scatter analysis
* Business storytelling — translating raw metrics into insight + recommendation pairs

\---

## 📁 Repository Contents

```
├── Airbnb\_Analysis\_Project.pbix
├── data/
│   ├── fact\_bookings.csv
│   ├── dim\_date.csv
│   ├── dim\_host.csv
│   ├── dim\_location.csv
│   └── dim\_property.csv
└── README.md
```

Open `Airbnb\_Analysis\_Project.pbix` in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free) to explore the dashboard interactively, or check the screenshots above for a quick static view without installing anything.

\---

## 📬 Contact

If you'd like to discuss this project or my approach to it, feel free to reach out via [LinkedIn](https://www.linkedin.com/in/kabeer-khan-analyst) or [email](mailto:kabeerk992@gmail.com).

