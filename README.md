# ☕ Campus Café Sales & Profitability Intelligence Dashboard

An executive Power BI solution built to analyze menu profitability, channel performance, and campus-wide sales dynamics for a multi-location campus dining operation.

<img width="566" height="316" alt="dashboard_screenshot" src="https://github.com/user-attachments/assets/1dfc8870-34a0-4389-85a9-df5db993418e" />

---

## 🎯 Executive Summary & Commercial Analysis

### Stakeholder Scope
Campus Dining General Manager, Head of Operations, and Retail Strategy Leads

### Key Operational Findings
* **Category Profitability Divergence:** Meals represent the primary top-line revenue driver at **£1,004.00**, but generate only **£279.42** in net profit, yielding a below-target margin of **27.8%**. In contrast, Drinks generate **£810.50** at a **62.1%** margin (**£503.32** net profit).
* **Channel Distribution:** In-person Counter transactions generate **62.99%** of total turnover (**£1.40K**), while digital App orders represent **37.01%** (**£0.82K**).
* **Menu Engineering Matrix:**
  * **Stars (High Volume, High Margin):** *Filter Coffee* (~£290 sales, ~62% margin) and *Cold Coffee / Fresh Juice* (~£215 sales, ~62% margin) drive sustainable earnings.
  * **Plowhorses / Workhorses (High Volume, Low Margin):** *Biryani* (~£380 sales, ~22% margin) and *Veg Thali* (~£235 sales, ~24% margin) capture high footfall but dilute blended profitability.
* **Campus Performance:** *North Campus* leads overall gross revenue at **£846.00**, outperforming *Hostel* (**£690.25**) and *South Campus* (**£688.75**).

### Commercial Risk & Margin Dilution
* **Unbundled Sales Exposure:** High transaction volume in low-margin food items without beverage attachments pulls aggregate gross margins below the baseline target of **44%**. 
* **Operational Bottlenecks:** Physical counter concentration (63%) increases service wait times during peak rush periods and relies entirely on manual cashier upselling, leading to low beverage attachment rates on meal tickets.

### Strategic & Prescriptive Recommendations
* **Automated Combo Bundling:** Deploy automated prompts within the ordering application to cross-sell a high-margin beverage (*Filter Coffee* or *Masala Tea*) alongside any Meal for an incremental £1.20 add-on.
* **Product Cost Re-Engineering:** Adjust supplier costs, reduce ingredient waste, or implement a targeted 5–8% price optimization on *Biryani* to elevate unit margins above 30%.
* **Data Hygiene Flag:** May represents an incomplete reporting cycle (**£26.50** across 420 total term orders) and has been excluded from baseline monthly run-rate calculations.
* **Target Success KPIs:** Elevate blended meal category margins from **27.8% to 34%+** and increase digital App adoption from **37% to 50%** over the upcoming academic term.

---

## 📊 Business Logic & Metric Dictionary

| Metric Name | Value / Format | Underlying Logic / Calculation | Business Purpose |
| :--- | :--- | :--- | :--- |
| **Total Sales** | `2.23K` (£2,225.00) | `SUM(campus_cafe_project[Total Sales])` | Monitors gross top-line revenue volume. |
| **Total Profit** | `£980.20` | `SUM(campus_cafe_project[Profit])` | Bottom-line earnings after product direct costs. |
| **Profit Margin** | `44%` | `DIVIDE([Total Profit], [Total Sales], BLANK())` | Measures operational profitability and cost efficiency. |
| **Total Orders** | `420` | `DISTINCTCOUNT(campus_cafe_project[Order_ID])` | Tracks total transaction volume and customer throughput. |
| **App Share** | `37.01%` | `DIVIDE(CALCULATE([Total Sales], Channel="App"), [Total Sales])` | Tracks digital channel penetration against physical counter volume. |

---

## 🛠️ Data Architecture & Implementation

### Semantic Data Model
* **Fact Table:** `campus_cafe_project`
* **Granularity:** Order transaction level containing `Order_ID`, `Order_Date`, `Item`, `Quantity`, `Total Sales`, `Time_Slot`, and `Payment`.
* **Dimensional Slicing:** `Category` (Drinks, Meals, Snacks), `Campus` (Hostel, North, South), and `Channel` (App, Counter).

### Core DAX Measures

```dax
// Base Aggregations
Total Sales = 
SUM('campus_cafe_project'[Total Sales])

Total Profit = 
SUM('campus_cafe_project'[Profit])

Total Orders = 
DISTINCTCOUNT('campus_cafe_project'[Order_ID])

// Performance Ratios
Profit Margin % = 
DIVIDE(
    [Total Profit], 
    [Total Sales], 
    BLANK()
)

App Share % = 
DIVIDE(
    CALCULATE([Total Sales], 'campus_cafe_project'[Channel] = "App"),
    [Total Sales],
    BLANK()
)

// Dynamic BCG Menu Matrix Classification
Menu Item Classification = 
VAR AvgMargin = CALCULATE([Profit Margin %], ALLSELECTED('campus_cafe_project'))
VAR MedianSales = 193.75 // Median revenue benchmark across active menu catalog
VAR CurrentMargin = [Profit Margin %]
VAR CurrentSales = [Total Sales]
RETURN
    SWITCH(
        TRUE(),
        CurrentSales >= MedianSales && CurrentMargin >= AvgMargin, "Star (High Sales, High Margin)",
        CurrentSales >= MedianSales && CurrentMargin < AvgMargin, "Workhorse (High Sales, Low Margin)",
        CurrentSales < MedianSales && CurrentMargin >= AvgMargin, "Puzzle (Low Sales, High Margin)",
        "Underperformer (Low Sales, Low Margin)"
    )
               
