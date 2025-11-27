Campus Recruitment Analysis using AWS Glue, Athena & S3

A complete end-to-end data engineering project built using Amazon S3, AWS Glue, AWS Athena, and SQL to analyze campus recruitment and placement trends.

🚀 Project Overview

This project automates the ingestion, cleaning, transformation, and querying of campus placement data to generate meaningful insights such as:

-->Company-wise selection count

-->College selection percentage

-->Year-wise & quarter-wise placement trends

-->College ranking based on total placements


The pipeline is fully serverless and scalable using AWS native services.

🏗 Architecture

S3 (raw → curated → processed)
       │
       ▼
AWS Glue Crawler 1 → Glue ETL Job → Clean data + Dim & Fact tables → processed/
       │
       ▼
AWS Glue Crawler 2 → Athena Database & Tables
       │
       ▼
AWS Athena Queries → Insights & Analysis

📁 S3 Bucket Structure

s3://campus-recruitment-data/
│
├── raw/ # Source CSV files
├── curated/ # Cleaned and transformed datasets
└── processed/ # Final Dim_Date, Dim_College, Fact_Campus tables

🔧 AWS Glue Setup

Crawler 1

Crawls raw/ folder

Generates tables

ETL script cleans data, creates:

dim_date

dim_college

fact_campus


Outputs to processed/


Crawler 2

Crawls processed/

Creates schema for Athena queries

🗄 Data Model

The project follows a Dim-Fact schema:

📌 Dimension Tables

dim_date

dim_college

dim_campus

📌 Fact Table

fact_campus


🔍 Analytical Queries (Athena)

1️⃣ Total Students Selected Per Company

SELECT
    f.company_name,
    SUM(f.student_selected) AS total_selected
FROM fact_campus f
GROUP BY f.company_name
ORDER BY total_selected DESC;


2️⃣ Overall Selection Percentage Per College (Year-wise)

SELECT
    year(date_parse(f.full_date, '%e-%b-%y')) AS year,
    f.college_name,
    SUM(CAST(f.student_selected AS DOUBLE)) / NULLIF(SUM(f.student_participated), 0) AS overall_percentage_selection
FROM fact_campus f
GROUP BY year(date_parse(f.full_date, '%e-%b-%y')), f.college_name
ORDER BY year, overall_percentage_selection DESC;


3️⃣ Students Participated vs Placed Per College (Year-wise)

SELECT
    year(date_parse(f.full_date, '%e-%b-%y')) AS year,
    f.college_name,
    SUM(f.student_participated) AS total_participated,
    SUM(f.student_selected) AS total_placed
FROM fact_campus f
LEFT JOIN dim_date d
ON f.date_key = d.date_key
GROUP BY year(date_parse(f.full_date, '%e-%b-%y')), f.college_name
ORDER BY year, total_placed DESC;


4️⃣ Quarter-wise Placement Analysis

SELECT
    year(date_parse(f.full_date, '%e-%b-%y')) AS year,
    quarter(date_parse(f.full_date, '%e-%b-%y')) AS quarter,
    f.college_name,
    SUM(f.student_selected) AS total_selected,
    SUM(f.student_participated) AS total_participated
FROM fact_campus f
LEFT JOIN dim_college c
    ON f.college_name = c.college
GROUP BY year(date_parse(f.full_date, '%e-%b-%y')),
         quarter(date_parse(f.full_date, '%e-%b-%y')),
         f.college_name
ORDER BY year, quarter, f.college_name;

5️⃣ College Ranking Year-wise Based on Total Placements

WITH placement_summary AS (
    SELECT
        d.fb_year AS year,
        f.college_name,
        SUM(f.student_selected) AS total_placed
    FROM fact_campus f
    LEFT JOIN dim_date d
        ON f.date_key = d.date_key
    GROUP BY
        d.fb_year,
        f.college_name
)
SELECT
    year,
    college_name,
    ROW_NUMBER() OVER (
        PARTITION BY year
        ORDER BY total_placed DESC
    ) AS ranking
FROM placement_summary
ORDER BY year, ranking;

🏁 Outcome

-->Built a scalable, serverless analytics pipeline

-->Created automated ETL using AWS Glue

-->Used Athena to deliver insights with fast, cost-effective SQL queries

-->Modeled data into Dim–Fact architecture

-->Improved decision-making for campus recruitment patterns

