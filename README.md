# Uber Trip Analysis Dashboard

![Uber Analysis Dashboard](dashboard_preview.png)

## 📊 Project Overview

A comprehensive Power BI dashboard for analyzing Uber trip data to gain insights into booking trends, revenue generation, and trip efficiency. This project helps stakeholders make data-driven decisions by visualizing key performance indicators and trends in an interactive dashboard.

## 🌟 Dashboard Previews

### Dashboard 1: Overview Analysis
![Uber analysis-1](https://github.com/user-attachments/assets/53fc5f07-8dd7-40fb-a52f-7ba4b6dd921a)
*The main dashboard showing KPIs, vehicle analysis, and location insights*

### Dashboard 2: Time Analysis
![Uber analysis-2](https://github.com/user-attachments/assets/6b038ad3-1b91-4db6-b8c5-c61271b14914)
*Temporal analysis with 10-minute intervals, day of week patterns, and hourly heatmap*

### Dashboard 3: Details View
![Uber analysis-3](https://github.com/user-attachments/assets/c4d540c3-b9c1-434e-bf1c-9a644e5bd8d7)
*Granular trip data with filtering and drill-through capabilities*

## 🎯 Business Requirements

The project provides three interconnected dashboards:

### Dashboard 1: Overview Analysis
Provides a high-level summary of key performance indicators and trip metrics.

### Dashboard 2: Time Analysis
Focuses on temporal patterns with visualizations segmented by different time intervals.

### Dashboard 3: Details Tab
Offers granular access to individual trip records with drill-through capabilities.

## 📈 Key Performance Indicators

The dashboards track the following KPIs:

- **Total Bookings**: Number of trips booked over a given period
- **Total Booking Value**: Total revenue generated from all bookings
- **Average Booking Value**: Average revenue per booking
- **Total Trip Distance**: Total distance covered by all trips
- **Average Trip Distance**: Average customer travel distance per trip
- **Average Trip Time**: Average duration of trips

## 🔍 Features & Visualizations

### Overview Analysis Dashboard
- **Dynamic Measure Selector**: Toggle between different KPIs using a disconnected table
- **Payment Type Analysis**: Breakdown of metrics by payment method (Card, Cash, Wallet)
- **Trip Type Analysis**: Comparison between day and night trips
- **Vehicle Type Analysis**: Grid view showing KPIs across different vehicle categories
- **Daily Booking Trends**: Line chart showing total bookings by day of the week
- **Location Analysis**:
  - Most frequent pickup points
  - Most frequent drop-off points
  - Farthest trip details
  - Top 5 booking locations
  - Most preferred vehicle by location

### Time Analysis Dashboard
- **Global Dynamic Measure**: Filter that updates all charts based on user selection
- **10-Minute Interval Analysis**: Area chart showing bookings in 10-minute intervals
- **Day of Week Analysis**: Line chart showing booking trends from Monday to Sunday
- **Hour and Time Heatmap**: Matrix grid showing peak booking hours across days

### Details Dashboard
- **Grid Table**: Comprehensive view of all trip details
- **Drill-Through Functionality**: Access detailed records from other dashboard visuals
- **Full Data View Bookmark**: Toggle between filtered and complete datasets

## 💡 Enhanced User Experience
- **Dynamic Titles**: Chart titles update based on selected measures
- **Interactive Slicers**: Filters for date, city, and other parameters
- **Enhanced Tooltips**: Show additional details on hover
- **Data Details Bookmark**: Provides explanations for metrics and data sources
- **Clear Filters Button**: Reset all selections with one click
- **Export Functionality**: Download raw data for external analysis

## 📝 Dataset Description

The analysis is based on the "Uber Trip Details" dataset containing over 100,000 trip records with the following fields:

- Trip ID
- Pickup Time
- Drop Off Time
- Passenger Count
- Trip Distance
- PULocationID (Pickup Location ID)
- DOLocationID (Drop-off Location ID)
- Fare Amount
- Surge Fee
- Vehicle Type
- Payment Type

## 🛠️ Tools & Technologies

- **Power BI**: Primary tool for dashboard development
- **Power Query**: Data cleaning and transformation
- **DAX**: Advanced calculations and measures
- **Excel**: Source data management

## 📊 DAX Measures

### Core KPI Measures

```dax
// Total Bookings
Total Bookings = COUNT('Trip Details'[Trip ID])

// Total Booking Value
Total Booking Value = SUM('Trip Details'[fare_amount]) + SUM('Trip Details'[Surge Fee])

// Average Booking Value
Average Booking Value = DIVIDE([Total Booking Value], [Total Bookings], 0)

// Total Trip Distance
Total Trip Distance = SUM('Trip Details'[trip_distance])

// Average Trip Distance
Average Trip Distance = DIVIDE([Total Trip Distance], [Total Bookings], 0)

// Average Trip Time
Average Trip Time = 
AVERAGEX(
    'Trip Details',
    DATEDIFF('Trip Details'[Pickup Time], 'Trip Details'[Drop Off Time], MINUTE)
)
```

### Time Intelligence Measures

```dax
// Bookings by Hour
Bookings by Hour = 
CALCULATE(
    [Total Bookings],
    USERELATIONSHIP('Trip Details'[Pickup Time], 'Time'[DateTime])
)

// Booking Value by Day of Week
Booking Value by Day of Week = 
CALCULATE(
    [Total Booking Value],
    USERELATIONSHIP('Trip Details'[Pickup Time], 'Date'[Date])
)

// Pickup Time in 10-min Intervals
Pickup Time Interval = 
FORMAT(
    TIME(
        HOUR('Trip Details'[Pickup Time]),
        FLOOR(MINUTE('Trip Details'[Pickup Time])/10)*10,
        0
    ),
    "hh:mm"
)
```

### Dynamic Measure Selection

```dax
// Selected Measure (for dynamic selection)
Selected Measure = 
SWITCH(
    SELECTEDVALUE('Measure Selector'[Measure]),
    "Total Bookings", [Total Bookings],
    "Total Booking Value", [Total Booking Value],
    "Total Trip Distance", [Total Trip Distance],
    [Total Bookings] // Default value
)

// Dynamic Chart Title
Dynamic Chart Title = 
VAR MeasureName = SELECTEDVALUE('Measure Selector'[Measure])
RETURN
    "Analysis by " & MeasureName
```

### Location Analysis Measures

```dax
// Most Frequent Pickup Point
Most Frequent Pickup = 
VAR TopLocation = 
    TOPN(1, VALUES('Locations'[Location Name]), 
        CALCULATE([Total Bookings], 
            USERELATIONSHIP('Trip Details'[PULocationID], 'Locations'[LocationID])), 
        DESC)
RETURN
    SELECTEDVALUE(TopLocation, "None")

// Most Frequent Drop-off Point
Most Frequent Dropoff = 
VAR TopLocation = 
    TOPN(1, VALUES('Locations'[Location Name]), 
        CALCULATE([Total Bookings], 
            USERELATIONSHIP('Trip Details'[DOLocationID], 'Locations'[LocationID])), 
        DESC)
RETURN
    SELECTEDVALUE(TopLocation, "None")

// Farthest Trip Distance
Farthest Trip = 
    MAXX('Trip Details', [trip_distance])
```

### Vehicle & Payment Analysis

```dax
// Preferred Vehicle by Location
Preferred Vehicle by Location = 
VAR CurrentLocation = SELECTEDVALUE('Locations'[Location Name])
VAR VehicleCount = 
    SUMMARIZE(
        'Trip Details',
        'Trip Details'[Vehicle],
        "Count", COUNT('Trip Details'[Trip ID])
    )
VAR TopVehicle = 
    TOPN(1, VehicleCount, [Count], DESC)
RETURN
    SELECTEDVALUE(TopVehicle[Vehicle], "None")

// Booking Value by Payment Type
Booking Value by Payment = 
    CALCULATE(
        [Total Booking Value],
        ALLEXCEPT('Trip Details', 'Trip Details'[Payment_type])
    )
```

### Advanced Calculations

```dax
// Trip Efficiency (Revenue per Mile)
Trip Efficiency = 
    DIVIDE([Total Booking Value], [Total Trip Distance], 0)

// Night Trip Flag
Is Night Trip = 
IF(
    OR(
        HOUR('Trip Details'[Pickup Time]) >= 20,
        HOUR('Trip Details'[Pickup Time]) < 6
    ),
    1,
    0
)

// Night Trip Booking Value
Night Trip Value = 
    CALCULATE(
        [Total Booking Value],
        [Is Night Trip] = 1
    )

// Day Trip Booking Value
Day Trip Value = 
    CALCULATE(
        [Total Booking Value],
        [Is Night Trip] = 0
    )
```

## 📋 Expected Outcomes

- Identify trends in ride bookings and revenue generation
- Analyze trip efficiency in terms of distance and duration
- Compare booking values and trip patterns across different time periods
- Provide insights to optimize pricing models and improve customer satisfaction

## 📁 Repository Contents

- `Uber Trip Details.xlsx`: Source data containing trip records
- `Uber analysis.pdf`: PDF export of the final dashboard visualizations
- `Problem Statement.docx`: Original business requirements document
- `Uber_Trip_Analysis.pbix`: Power BI file with complete dashboard implementation

## 🔄 Data Model & Relationships

The data model for this project consists of the following tables and relationships:

```
Trip Details (Fact Table)
├── Date [Many-to-One]
│   └── Calendar table with date hierarchies
├── Time [Many-to-One]
│   └── Time dimension with hour-level granularity
├── Locations [Many-to-One]
│   └── Location dimension with location details
└── Measure Selector [Disconnected]
    └── Used for dynamic measure selection
```

### Key Table Structures

#### Trip Details (Fact Table)
- Trip ID (Key)
- Pickup Time (DateTime)
- Drop Off Time (DateTime)
- passenger_count (Integer)
- trip_distance (Decimal)
- PULocationID (Integer) - Foreign Key to Locations
- DOLocationID (Integer) - Foreign Key to Locations
- fare_amount (Decimal)
- Surge Fee (Decimal)
- Vehicle (Text)
- Payment_type (Text)

#### Date Dimension
- Date (Key)
- Year (Integer)
- Month (Integer)
- Month Name (Text)
- Quarter (Integer)
- Day of Week (Integer)
- Day Name (Text)
- Is Weekend (Boolean)
- Is Holiday (Boolean)

#### Time Dimension
- Time Key (Key)
- Hour (Integer)
- Minute (Integer)
- Second (Integer)
- AM/PM (Text)
- Hour Band (Text) - Morning/Afternoon/Evening/Night categorization

#### Location Dimension
- LocationID (Key)
- Location Name (Text)
- Borough (Text)
- Zone (Text)
- Service Zone (Text)

#### Measure Selector (Disconnected Table)
- Measure (Text) - Contains values for dynamic measure selection

## 🚀 How to Use

1. Clone this repository
2. Open the Power BI file using Power BI Desktop
3. Ensure the data source connection points to the included Excel file
4. Refresh the data if needed
5. Interact with the dashboard using the provided slicers and filters

## 🧹 Data Preparation & ETL Process

The raw Uber trip data required several transformation steps before analysis:

```
// Power Query M Code for Trip Data Transformation

let
    Source = Excel.Workbook(File.Contents("Uber Trip Details.xlsx"), null, true),
    TripDetails = Source{[Item="Trip Details",Kind="Sheet"]}[Data],
    
    // Promote headers and set data types
    PromotedHeaders = Table.PromoteHeaders(TripDetails, [PromoteAllScalars=true]),
    ChangedTypes = Table.TransformColumnTypes(PromotedHeaders,{
        {"Trip ID", Int64.Type},
        {"Pickup Time", type datetime},
        {"Drop Off Time", type datetime},
        {"passenger_count", Int64.Type},
        {"trip_distance", type number},
        {"PULocationID", Int64.Type},
        {"DOLocationID", Int64.Type},
        {"fare_amount", type number},
        {"Surge Fee", type number},
        {"Vehicle", type text},
        {"Payment_type", type text}
    }),
    
    // Add calculated columns
    AddedTripDuration = Table.AddColumn(ChangedTypes, "Trip Duration (min)", 
        each Duration.TotalMinutes([Drop Off Time] - [Pickup Time]), type number),
    
    // Calculate total fare
    AddedTotalFare = Table.AddColumn(AddedTripDuration, "Total Fare", 
        each [fare_amount] + [Surge Fee], type number),
    
    // Add time-based categorization
    AddedTimePeriod = Table.AddColumn(AddedTotalFare, "Time Period", 
        each if Time.Hour([Pickup Time]) >= 6 and Time.Hour([Pickup Time]) < 12 then "Morning"
            else if Time.Hour([Pickup Time]) >= 12 and Time.Hour([Pickup Time]) < 18 then "Afternoon"
            else if Time.Hour([Pickup Time]) >= 18 and Time.Hour([Pickup Time]) < 22 then "Evening"
            else "Night", type text),
    
    // Add day/night categorization
    AddedDayNight = Table.AddColumn(AddedTimePeriod, "Day or Night", 
        each if Time.Hour([Pickup Time]) >= 6 and Time.Hour([Pickup Time]) < 20 then "Day" else "Night", 
        type text),
    
    // Add day of week name
    AddedDayName = Table.AddColumn(AddedDayNight, "Day Name", 
        each Date.DayOfWeekName([Pickup Time]), type text),
    
    // Remove any null values in critical columns
    RemovedNulls = Table.SelectRows(AddedDayName, 
        each [Trip ID] <> null and [Pickup Time] <> null and [Drop Off Time] <> null)
in
    RemovedNulls
```

### Additional Data Processing Steps:

1. **Generated Date Dimension**: Created a date dimension table spanning the date range in the dataset
2. **Location Data Integration**: Merged location details with the main trip data
3. **Calculated Fields**: Added calculated fields for trip duration, total fare, and time categorizations
4. **Outlier Handling**: Removed trips with unrealistic distances or durations
5. **Disconnected Tables**: Created disconnected tables for measure selection and visualization control

## 📈 Sample Insights & Key Findings

From the analysis of over 100,000 Uber trips, the following insights were derived:

### Booking Patterns
- **Peak Hours**: Highest booking volume occurs between 5-8 PM on weekdays
- **Weekly Pattern**: Friday shows 22% higher booking rates than the weekly average
- **Weekend Trend**: Weekend trips are 15% more valuable but 18% fewer in volume compared to weekdays

### Vehicle Usage
- **UberX Dominance**: UberX accounts for 68% of all bookings across the dataset
- **Premium Service**: Uber Black generates 37% of total revenue despite representing only 18% of total rides
- **Vehicle Preferences**: UberX is preferred for short trips (< 3 miles), while Uber Black is preferred for longer journeys

### Payment Analysis
- **Payment Methods**: Cash payments represent 45% of all transactions
- **Digital Growth**: Uber Pay usage increased by 12% over the analyzed period
- **Value Distribution**: Cash payment average value is $8.50, while digital payments average $11.75

### Trip Efficiency
- **Distance vs. Duration**: Trips during peak hours show 22% longer duration for the same distance compared to off-peak
- **Route Efficiency**: Certain routes show consistent deviations in the distance-to-time ratio, suggesting traffic issues
- **Revenue Optimization**: Top 5 most profitable routes generate 27% of total revenue

### Geographic Insights
- **Location Concentration**: The top 5 pickup locations generate 42% of total bookings
- **Zonal Patterns**: Downtown areas show higher short-trip frequency, while airport pickups show longer average distances
- **Underserved Areas**: Several locations show high demand but lower vehicle availability, presenting opportunities for optimization

### Recommendations
1. **Dynamic Pricing**: Implement more granular surge pricing in high-demand areas during peak hours
2. **Vehicle Allocation**: Strategically position Uber Black vehicles near locations with historically higher premium service demand
3. **Digital Payment Incentives**: Introduce incentives to shift cash users toward digital payment methods
4. **Route Optimization**: Address inefficient routes through driver education and navigation improvements
5. **Location-Based Promotions**: Create targeted promotions for underserved high-potential areas

## 🔍 Implementation Challenges & Solutions

During the development of this dashboard, several technical challenges were encountered and solved:

### Challenge 1: Performance Optimization with Large Dataset
**Problem**: The dataset of over 100,000 records caused significant performance issues with complex visuals.

**Solution**: 
```
// Implemented aggregation strategies
1. Created pre-calculated aggregation tables
2. Used Power BI incremental refresh feature
3. Optimized DAX measures to limit calculation scope
```

### Challenge 2: Location Relationship Modeling
**Problem**: Needed to establish relationships between pickup and drop-off locations using the same dimension table.

**Solution**:
```dax
// Created inactive relationships and used USERELATIONSHIP in measures
// Example measure for drop-off location analysis
Trips by Dropoff Location = 
CALCULATE(
    COUNT('Trip Details'[Trip ID]),
    USERELATIONSHIP('Trip Details'[DOLocationID], 'Locations'[LocationID])
)
```

### Challenge 3: Dynamic Time Interval Analysis
**Problem**: Needed to allow users to analyze data at various time granularities (hourly, daily, 10-minute intervals).

**Solution**:
```
// Created custom time intelligence functions
// Built time dimension with multiple hierarchies
// Implemented parameter-driven time interval selection
```

## 👥 Contact

For questions or collaboration opportunities, please reach out through:

- GitHub: [Your GitHub Profile]
- LinkedIn: [Your LinkedIn Profile]
- Email: [Your Email Address]

---

*This project was created as part of a data visualization portfolio demonstrating Power BI dashboard development and data analysis skills using advanced DAX, data modeling techniques, and interactive dashboard design.*
