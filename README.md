# Power BI Project: Booking Store Analysis

This Power BI project analyzes booking data from a multi-service business to provide insights into booking patterns, revenue generation, and customer behavior.

## Project Overview

**Purpose:** This project aims to clean, analyze, and visualize booking data to understand key performance indicators (KPIs) and trends within the business. The goal is to create an interactive dashboard that provides actionable insights for management regarding booking types, popular services, revenue streams, and instructor performance.

## Data Source(s)

The data for this project comes from a booking dataset containing the following columns:

*   **Booking ID:** Unique identifier for each booking.
*   **Customer ID:** Unique identifier for each customer.
*   **Customer Name:** Name of the customer.
*   **Booking Type:** Type of booking (e.g., Class, Facility Rental, Party).
*   **Booking Date:** Date of the booking.
*   **Status:** Status of the booking (e.g., Confirmed, Pending, Canceled).
*   **Class Type:** Type of class booked (e.g., Art, Gymnastics, Yoga).
*   **Instructor:** Name of the instructor for the class.
*   **Time Slot:** Time slot for the booking.
*   **Duration (mins):** Duration of the booking in minutes.
*   **Price:** Price of the booking.
*   **Facility:** Facility used for the booking.
*   **Theme:** Theme associated with the booking.
*   **Subscription Type:** Type of subscription (if applicable).  *Note: This column was excluded from analysis due to containing all NULL values.*
*   **Service Name:** Name of the service booked.
*   **Service Type:** Type of service booked.
*   **Customer Email:** Email address of the customer.
*   **Customer Phone:** Phone number of the customer.

The raw data was initially in a CSV file (`DataAnalyst_Assesment_Dataset.csv`) and imported into a SQL Server database for cleaning and preprocessing.

## Data Preparation

The following steps were performed using SQL to prepare the data for analysis:

*   **Database and Table Creation:**
    *   A database named "assessment" was created.
    *   A table named "bookings" was created with appropriate data types for each column.
*   **Data Import:** The data was imported from the CSV file into the "bookings" table using `BULK INSERT`.
*   **Missing Value Handling:**
    *   Missing `Facility` values were filled with the most frequently used facility for the corresponding `Booking_Type`.
    *   Remaining missing `Facility` values were filled with the corresponding `Service_Name`.
    *   NULL values in the `Theme` column were replaced with 'No theme'.
*   **Duplicate Removal:**  *(Implicitly handled during the loading process to the SQL table assuming the primary key constraint prevented duplicates)*
*   **Data Standardization:**
    *   `Status` values were standardized to "Confirmed" and "Pending."
    *   Unnecessary characters were removed from `Customer_Phone` numbers.
*   **Derived Column Creation:**
    *   A `Booking_Month_Year` column was created to extract the month and year from the `Booking_Date` for easier trend analysis.

The SQL queries used for data cleaning and preparation are:

```sql
-- Create Database
CREATE DATABASE assessment;

-- Create Table Bookings
CREATE TABLE bookings (
    Booking_ID VARCHAR(50) PRIMARY KEY,
    Customer_ID INT,
    Customer_Name NVARCHAR(100),
    Booking_Type NVARCHAR(50),
    Booking_Date DATE,
    Status NVARCHAR(50),
    Class_Type NVARCHAR(50),
    Instructor NVARCHAR(100),
    Time_Slot NVARCHAR(50),
    [Duration (mins)] INT,
    Price DECIMAL(10,2),
    Facility NVARCHAR(100),
    Theme NVARCHAR(100),
    Subscription_Type NVARCHAR(50),
    Service_Name NVARCHAR(100),
    Service_Type NVARCHAR(50),
    Customer_Email NVARCHAR(100),
    Customer_Phone NVARCHAR(20)
);

-- Data Insertion into Database Server
BULK INSERT bookings
FROM 'C:\Users\fayaz\Downloads\DataAnalyst_Assesment_Dataset.csv'
WITH (
    FORMAT = 'CSV',
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    TABLOCK
);

-- Handle Missing Values: Fill Missing Facility Based on Booking Type
UPDATE bookings
SET Facility = (
    SELECT TOP 1 Facility
    FROM bookings
    WHERE Booking_Type = bookings.Booking_Type AND Facility IS NOT NULL
    GROUP BY Facility
    ORDER BY COUNT(*) DESC
)
WHERE Facility IS NULL;

-- Updating Null Facility Values:
UPDATE bookings
SET Facility = Service_Name
WHERE Facility IS NULL;

-- Standardizing Status Values
UPDATE bookings
SET Status = 'Confirmed'
WHERE Status IN ('confirmed', 'CONFIRM', 'Confirm');

UPDATE bookings
SET Status = 'Pending'
WHERE Status IN ('PENDING', 'Pending', 'pend');

-- Cleaning Phone Numbers
UPDATE bookings
SET Customer_Phone = REPLACE(REPLACE(REPLACE(Customer_Phone, '-', ''), ' ', ''), '(', '');

-- Updating NULL VALUES in THEME
UPDATE bookings
SET Theme = 'No theme'
WHERE Theme IS NULL;

-- Create a Month-Year Column for Filtering in Power BI
ALTER TABLE bookings ADD Booking_Month_Year VARCHAR(20);
UPDATE bookings
SET Booking_Month_Year = FORMAT(Booking_Date, 'MMM yyyy');
Use code with caution.
Markdown
Report Design
The interactive dashboard consists of two pages:

Page 1: Booking Overview & Key Metrics

Key Performance Indicators (KPIs) - Card Visuals:

Total Revenue: Displays the total revenue generated from all bookings (SUM(Price)).

Total Bookings: Shows the total number of bookings (COUNT(Booking_ID)).

Average Revenue: Represents the average revenue per booking (AVERAGE(Price)).

Slicers (Dropdown Filters) for User Interaction:

Status: Allows filtering based on booking confirmation status (e.g., Pending, Confirmed, Canceled). Field Used: Status

Booking Month: Enables filtering bookings by month and year. Field Used: Booking_Month_Year (Measure)

Line Charts for Trend Analysis:

Count of Bookings Over Time: Tracks the number of bookings per month to identify demand trends. X-axis: Booking_Month_Year; Y-axis: COUNT(Booking_ID)

Price by Booking Month: Shows revenue fluctuations over different months. X-axis: Booking Date; Y-axis: SUM(Price)

Pie Chart for Booking by Theme:

Booking Count by Theme: Provides a visual breakdown of booking count by Theme. Values: COUNT(Booking_ID); Legend: Theme

Stacked Line Chart for Revenue Breakdown:

Revenue by Booking Type: Compares revenue distribution among different types of bookings (e.g., Classes, Facility Rentals, Subscriptions). X-axis: Booking Type; Y-axis: SUM(Price)

Clustered Bar Chart for Facility-Based Revenue:

Revenue by Facility: Highlights which facilities generate the most revenue. X-axis: Facility; Y-axis: SUM(Price)

Page 2: Booking Insights & Revenue Analysis

Donut Chart - Revenue by Service Type:

Displays the revenue distribution across different service types (e.g., Play area, Party room, Gymnastics, Dance, Art). Legend: Service_Type; Values: SUM(Price)

Clustered Bar Chart - Count of Bookings by Class Type:

Shows the total number of bookings for each class type (e.g., Art, Gymnastics, Yoga). X-axis: Class_Type; Y-axis: COUNT(Booking_ID)

Stacked Bar Chart - Revenue by Instructor:

Analyzes revenue contributions by each instructor, helping to evaluate their performance. X-axis: Instructor; Y-axis: SUM(Price)

Table - Instructor Performance:

Provides detailed insights into each instructor’s performance, including the number of bookings and total revenue generated. Fields: Instructor, Class_Type, COUNT(Booking_ID), SUM(Price)

Table - Service Details:

Displays available services along with their service type and associated themes. Fields: Service_Name, Service_Type, Theme

Key Learnings
This project provided valuable experience in:

Data cleaning and preprocessing using SQL.

Designing and implementing interactive dashboards in Power BI.

Identifying key performance indicators (KPIs) relevant to a multi-service business.

Visualizing data to reveal trends in bookings, revenue, and customer behavior.

Understanding the importance of data quality and consistency for accurate analysis.

Analyzing instructor performance and its impact on revenue.

Usage
This dashboard is designed to be used by:

Management: To gain a high-level overview of business performance, identify trends, and make strategic decisions.

Department Heads: To analyze performance within specific service areas (e.g., classes, facility rentals) and optimize resource allocation.

Instructors: To understand their individual performance and identify areas for improvement.

Marketing Team: To understand booking patterns and customer preferences, and tailor marketing campaigns accordingly.
