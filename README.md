# Bookings-Data-Engineer-Project

## Project Overview  
**Project Title**: Hotel Bookings Data Engineer Project  
**Database**: hotel_db

This project is an end‑to‑end Hotel Booking Analytics System built entirely in Snowflake, designed to simulate how a real hotel processes raw booking files, cleans messy data, transforms it into business‑ready tables, and generates insights for management. The pipeline follows the Medallion Architecture (Bronze → Silver → Gold) and includes everything from data ingestion and validation to advanced SQL transformations and dashboard creation.

The goal was to practice full‑stack data engineering skills such as building data pipelines, cleaning and standardizing inconsistent datasets, designing analytical models, and producing insights that support real business decisions.

<img width="679" height="295" alt="image" src="https://github.com/user-attachments/assets/e5984b66-cb29-4d3b-ac15-a91635ccda44" />

<img width="807" height="465" alt="Screenshot 2026-05-02 at 1 24 38 PM" src="https://github.com/user-attachments/assets/4de651e1-bfa2-4c0a-8f4c-3215fbc713cc" />


****Objectives****
---
1. **Build a Snowflake‑Native Data Pipeline**: Ingest raw hotel booking CSV files into Snowflake stages and load them into Bronze tables using file formats and COPY INTO.
2. **Clean and Standardize the Dataset (Silver Layer)**: Apply data‑quality rules to fix invalid dates, missing values, negative pricing, corrupted emails, typos, and duplicates.
3. **Transform Data for Analytics (Gold Layer)**: Create business‑ready tables by aggregating monthly revenue, monthly bookings, top cities, booking types, and booking statuses.
4. **Create an Interactive Dashboard in Snowsight**: Build visualizations such as monthly revenue, monthly bookings, top revenue‑generating cities, and booking type breakdowns, all directly inside Snowflake.

****Project Structure****
---

**1. Database Setup**




   - **Table Creation**: Designed and created tables for branches, employees, members, books, issued records, and return records, each with relevant columns, constraints, and relationships.

```sql
-- Create DataBase
CREATE DATABASE HOTEL_DB;

-- Create File Format
CREATE OR REPLACE FILE FORMAT FF_CSV
    TYPE = 'CSV'
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '')

-- Create Stage
CREATE OR REPLACE STAGE STG_HOTEL_BOOKINGS
-- FILE_FORMAT = (FORMAT_NAME = FF_CSV)
-- ON_ERROR = 'CONTINUE'
FILE_FORMAT = FF_CSV;

-- Create Table BRONZE_HOTEL_BOOKING
CREATE TABLE BRONZE_HOTEL_BOOKING (
    booking_id STRING,
    hotel_id STRING,
    hotel_city STRING,
    customer_id STRING,
    customer_name STRING,
    customer_email STRING,
    check_in_date STRING,
    check_out_date STRING,
    room_type STRING,
    num_guests STRING,
    total_amount STRING,
    currency STRING,
    booking_status STRING
);

-- Loading Data from Stage to Bronze Table
COPY INTO BRONZE_HOTEL_BOOKING
FROM @STG_HOTEL_BOOKINGS
FILE_FORMAT = (FORMAT_NAME = FF_CSV)
ON_ERROR = 'CONTINUE';

SELECT * FROM BRONZE_HOTEL_BOOKING LIMIT 50;

-- Create Table SILVER_HOTEL_BOOKINGS
CREATE TABLE SILVER_HOTEL_BOOKINGS (
    booking_id VARCHAR,
    hotel_id VARCHAR,
    hotel_city VARCHAR,
    customer_id VARCHAR,
    customer_name VARCHAR,
    customer_email VARCHAR,
    check_in_date DATE,
    check_out_date DATE,
    room_type VARCHAR,
    num_guests INTEGER,
    total_amount FLOAT,
    currency VARCHAR,
    booking_status VARCHAR
);

-- Checking for errors
SELECT customer_email
FROM BRONZE_HOTEL_BOOKING
WHERE NOT (customer_email LIKE '%@%.%')
    OR customer_email IS NULL

SELECT total_amount
FROM BRONZE_HOTEL_BOOKING
WHERE TRY_TO_NUMBER(total_amount) < 0;

SELECT check_in_date, check_out_date
FROM BRONZE_HOTEL_BOOKING
WHERE TRY_TO_DATE(check_out_date) < TRY_TO_DATE(check_in_date);

SELECT DISTINCT booking_status
FROM BRONZE_HOTEL_BOOKING;

-- Iserting Cleaned data to Silver layer
INSERT INTO SILVER_HOTEL_BOOKINGS
SELECT
    booking_id,
    hotel_id,
    INITCAP(TRIM(hotel_city)) AS hotel_city,
    customer_id,
    INITCAP(TRIM(customer_name)) AS customer_name,
    CASE
        WHEN customer_email LIKE '%@%.%' THEN LOWER(TRIM(customer_email))
        ELSE NULL
    END AS customer_email,
    TRY_TO_DATE(NULLIF(check_in_date, '')) AS check_in_date,
    TRY_TO_DATE(NULLIF(check_out_date, '')) AS check_out_date,
    room_type,
    num_guests,
    ABS(TRY_TO_NUMBER(total_amount)) AS total_amount,
    currency,
    CASE
        WHEN LOWER(booking_status) in ('confirmeeed', 'confirmd') THEN 'Confirmed'
        ELSE booking_status
    END AS booking_status
    FROM BRONZE_HOTEL_BOOKING
    WHERE
        TRY_TO_DATE(check_in_date) IS NOT NULL
        AND TRY_TO_DATE(check_out_date) IS NOT NULL
        AND TRY_TO_DATE(check_out_date) >= TRY_TO_DATE(check_in_date);

SELECT * FROM SILVER_HOTEL_BOOKINGS LIMIT 30;

-- Create Gold Tables
CREATE TABLE GOLD_AGG_DAILY_BOOKING AS
SELECT
    check_in_date AS date,
    COUNT(*) AS total_booking,
    SUM(total_amount) AS total_revenue
FROM SILVER_HOTEL_BOOKINGS
GROUP BY check_in_date
ORDER BY date;

CREATE TABLE GOLD_AGG_HOTEL_CITY_SALES AS
SELECT
    hotel_city,
    SUM(total_amount) AS total_revenue
FROM SILVER_HOTEL_BOOKINGS
GROUP BY hotel_city
ORDER BY total_revenue DESC;

CREATE TABLE GOLD_BOOKING_CLEAN AS
SELECT
    booking_id,
    hotel_id,
    hotel_city,
    customer_id,
    customer_name,
    customer_email,
    check_in_date,
    check_out_date,
    room_type,
    num_guests,
    total_amount,
    currency,
    booking_status
FROM SILVER_HOTEL_BOOKINGS;

SELECT * FROM GOLD_AGG_DAILY_BOOKING LIMIT 30;

SELECT * FROM GOLD_AGG_HOTEL_CITY_SALES LIMIT 30;
```


  
****Conclusion****
---

This project highlights my ability to build a production‑style data pipeline using Snowflake. By ingesting raw booking files, cleaning and standardizing messy data, transforming it into analytics‑prepped tables, and creating an interactive dashboard, I demonstrated skills in both data engineering and analysis, turning unstructured booking records into insights that can be utilized for business decisions

















