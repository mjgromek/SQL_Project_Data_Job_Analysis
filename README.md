# Introduction
This project explores the job market for Data Analysts in Western Europe using SQL.
By querying a dataset of job postings, it identifies where the best opportunities are and which skills are worth investing time in.

Specifically, the analysis focuses on:

Top-paying Data Analyst roles in the Netherlands, Belgium, and Germany (with an emphasis on remote jobs and postings that include salary data).

Skills required for those top-paying roles, showing which tools and technologies are most commonly requested.

Most in-demand skills for Data Analysts in the Netherlands, based on how often each skill appears in job descriptions.

“Optimal” skills to learn, combining both demand (how often a skill appears) and average salary across the Netherlands, Belgium, and Germany.

The goal of this project is to give aspiring or current Data Analysts clear, data-driven guidance on which skills to prioritize to maximize both employability and earning potential.

SQL Queries: [project_sql] (/project_sql/)

# Background

Trying to figure out which data-analytics skills actually pay off can be confusing, especially across different countries. I built this project to cut through that noise by using SQL to uncover the highest-paying Data Analyst roles and the skills that show up most often in real job postings for the Netherlands, Belgium, and Germany. The goal is to give myself (and others) a clear, data-driven roadmap of what to learn next.

The dataset comes from Luke Barousse’s SQL Course (https://lukebarousse.com/sql), which includes rich information on job titles, salaries, locations, and required skills—perfect for exploring the European data analytics job market.

### The question I wanted to answer through my SQL queries were

1. What are the top-paying data analyst jobs in Germany, Netherlands and Belgium?
2. What skills are required for the top-paying data analyst jobs in the Netherlands and Belgium?
3. What are the most in-demand skills for data analysts in the Netherlands?
4. What are the most in-demand skills for data analysts in the Netherlands?
5. What are the most optimal skills to learn in the Netherland, Germany and Belgium? 

# Tools I Used

Tools I Used
For my deep dive into the data analyst job market in the Netherlands, Belgium, and Germany, I leaned on a few key tools:

- **SQL** – The core of this project, used to query the job postings dataset and uncover trends in salaries, locations, and skills.
- **PostgreSQL** – The database engine powering the analysis, handling all tables for job postings, companies, and skills.
- **Visual Studio Code** – My main workspace for writing, running, and iterating on SQL queries.
- **Git & GitHub** – Used for version control and sharing the project, keeping track of query changes, results, and documentation over time.

# The Analysis

Each query in this project is designed to explore a different angle of the Data Analyst job market in the Netherlands, Belgium, and Germany. From identifying the highest-paying roles to uncovering which skills are most in demand and which offer the best salary–demand balance, every step builds on the last to answer one core question: what should an aspiring Data Analyst focus on to maximize opportunities and earning potential?

### 1. Top Paying Data Analyst Jobs

To uncover the highest–paying opportunities, I filtered data analyst roles by yearly salary and location, focusing on positions based in the Netherlands, Belgium, and Germany that include explicit salary information. This query surfaces the top 10 best-paid data analyst jobs, highlighting where the most lucrative roles are and which companies are offering them.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM 
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short LIKE '%Data%' AND
    job_title_short LIKE '%Analyst%' AND
    (job_location LIKE '%Netherlands%' OR
    job_location LIKE '%Belgium%') AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
```

|  Job ID | Job Title                       | Location                 | Schedule  | Salary (€/year) | Posted Date         | Company        |
| ------: | ------------------------------- | ------------------------ | --------- | --------------: | ------------------- | -------------- |
|   24675 | Staff Research Engineer         | Amsterdam, Netherlands   | Full-time |         177,283 | 2023-06-21 22:25:43 | ServiceNow     |
|  155094 | Data Architect                  | Machelen, Belgium        | Full-time |         165,000 | 2023-07-26 17:12:05 | Devoteam       |
|   88304 | Data Architect                  | Zaventem, Belgium        | Full-time |         165,000 | 2023-07-12 07:50:13 | Devoteam       |
|  280829 | Data Architect                  | Machelen, Belgium        | Full-time |         165,000 | 2023-08-03 15:29:20 | Devoteam       |
|   53650 | Data Architect                  | Brussels, Belgium        | Full-time |         165,000 | 2023-01-18 06:27:33 | Blue Harvest   |
|  539907 | Data Architect                  | Anderlecht, Belgium      | Full-time |         163,782 | 2023-06-22 17:55:14 | Ypto           |
| 1719883 | Data Architect - Sustainability | Netherlands              | Full-time |         155,000 | 2023-12-21 13:38:35 | LyondellBasell |
| 1051496 | Analytics Engineer (L5) - EMEA  | Amsterdam, Netherlands   | Full-time |         147,500 | 2023-07-13 18:16:22 | Netflix        |
|  348471 | Data Analyst, Reporting         | Amsterdam, Netherlands   | Full-time |         111,202 | 2023-01-02 08:29:05 | Adyen          |
|  250812 | C002927 Data Analyst (NS)       | Braine-l'Alleud, Belgium | Full-time |         111,175 | 2023-07-10 16:54:56 | EMW, Inc.      |

Key Insights:

- The highest salaries (up to ~€177k) are associated with very senior roles like Staff Research Engineer and Data Architect, rather than classic “junior” Data Analyst titles.
- Belgium has several high-paying architect roles (Devoteam, Blue Harvest, Ypto), while the Netherlands features big tech names like ServiceNow, Netflix, Adyen, and LyondellBasell.
- For analysts, this suggests a natural progression: moving towards data architecture / analytics engineering can significantly increase earning potential.

### 2. Skills for Top-Paying Data Analyst Jobs

To understand which skills are most valuable at the very top of the market, I first isolated the 10 highest-paying Data Analyst roles in the Netherlands and Belgium. I then joined those roles to the skills tables to see exactly which tools and technologies employers expect for these premium positions. This query reveals the skills that consistently appear in the best-paid data analyst jobs.

```sql
WITH top_paying_jobs AS (
SELECT
    job_id,
    job_title,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM 
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short LIKE '%Data%' AND
    job_title_short LIKE '%Analyst%' AND
    (job_location LIKE '%Netherlands%' OR
    job_location LIKE '%Belgium%') AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10
)

SELECT 
    skills,
    top_paying_jobs.*
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY 
    salary_year_avg
LIMIT 10
```

| Skill      | Job Title               | Company  | Salary (€/year) |
| ---------- | ----------------------- | -------- | --------------: |
| databricks | Azure Data Analyst      | Devoteam |         111,175 |
| ssis       | Azure Data Analyst      | Devoteam |         111,175 |
| go         | Azure Data Analyst      | Devoteam |         111,175 |
| azure      | Azure Data Analyst      | Devoteam |         111,175 |
| aws        | Azure Data Analyst      | Devoteam |         111,175 |
| power bi   | Azure Data Analyst      | Devoteam |         111,175 |
| sql        | Azure Data Analyst      | Devoteam |         111,175 |
| python     | Azure Data Analyst      | Devoteam |         111,175 |
| sql server | Azure Data Analyst      | Devoteam |         111,175 |
| excel      | Data Analyst, Reporting | Adyen    |         111,202 |


Key Insights:

- High-paying roles blend cloud platforms (Azure, AWS) with data engineering tools (Databricks, SSIS) and core analytics skills (SQL, Python).
- Power BI and Excel still appear in premium roles, showing that storytelling and reporting skills remain important even as tech stacks become more advanced.
- Top-paying “analyst” roles look a lot like analytics engineer positions: strong mix of BI, programming, databases, and cloud.

### 3. Most In-Demand Skills for Data Analysts in the Netherlands

To see which skills are requested most often, I counted how many times each skill appears in job postings based in the Netherlands. By grouping by skill and ordering by the count, this query highlights the technologies and tools that are most commonly required for data roles in this region.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_location LIKE '%Netherlands%'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 10
```
| Skill      | Demand Count |
| ---------- | -----------: |
| python     |       10,018 |
| sql        |        9,178 |
| azure      |        5,926 |
| r          |        2,989 |
| aws        |        2,688 |
| spark      |        2,495 |
| databricks |        2,297 |
| power bi   |        2,279 |
| tableau    |        1,924 |
| excel      |        1,852 |

Key Insights:

- Python and SQL are the clear core skills for data roles in the Netherlands.
- Cloud skills (Azure, AWS) and big data tools (Spark, Databricks) show strong demand, reflecting modern data stacks.
- BI & visualization tools (Power BI, Tableau, Excel) remain essential for delivering insights to stakeholders.

pie title Top 10 In-Demand Skills (Netherlands)
  "Python" : 10018
  "SQL" : 9178
  "Azure" : 5926
  "R" : 2989
  "AWS" : 2688
  "Spark" : 2495
  "Databricks" : 2297
  "Power BI" : 2279
  "Tableau" : 1924
  "Excel" : 1852

### 4. Highest-Paying Skills for Data Analysts in the Netherlands

To understand which skills are associated with the best-paying data analyst roles, I filtered job postings for Data Analyst positions in the Netherlands that include salary data. Then, I calculated the average yearly salary for each skill. This query highlights which skills tend to command the highest pay in the Dutch data analyst job market.

```sql
SELECT 
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE 
    job_title_short LIKE '%Data%' AND 
    job_title_short LIKE '%Analyst%' AND 
    job_location LIKE '%Netherlands%' AND
    salary_year_avg IS NOT NULL 
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

| Skill      | Avg Salary (€/year) |
| ---------- | ------------------: |
| nosql      |             177,283 |
| aws        |             177,283 |
| azure      |             177,283 |
| hadoop     |             144,243 |
| sap        |             123,725 |
| spark      |             118,486 |
| vba        |             111,202 |
| airflow    |             111,202 |
| pyspark    |             111,189 |
| word       |             111,175 |
| outlook    |             108,088 |
| scala      |             107,250 |
| tableau    |             106,959 |
| powerpoint |             105,000 |
| ms access  |             105,000 |
| go         |             103,896 |
| looker     |             103,826 |
| r          |              99,956 |
| c#         |              98,500 |
| jira       |              98,500 |
| qlik       |              98,500 |
| sql        |              96,006 |
| matlab     |              90,838 |
| cognos     |              89,204 |
| sas        |              89,204 |

Key Insights:

- Top-paying skills cluster around cloud and big data: NoSQL, AWS, Azure, Hadoop, Spark, Airflow, PySpark.
- Enterprise tools (SAP, Tableau, Qlik, Looker, Cognos, SAS) are also associated with strong salaries.
- Core analysis tools like SQL and R have slightly lower average salaries, likely because they appear in a broader mix of mid-level and senior roles.
- Even “office” tools (Word, PowerPoint, Outlook, MS Access, VBA) show high averages because they attach to a few extremely well-paid senior roles that list them alongside advanced tech skills.

### 5. Optimal Skills to Learn

To find the “best bang for your buck” skills, I combined both demand and pay across the Netherlands, Germany, and Belgium. First, I measured how often each skill appears in job postings (demand). Then, I calculated the average salary for Data Analyst roles requiring those skills. By joining these results and filtering to skills with at least 10 postings, this query surfaces the skills that are both high-paying and widely requested—making them strong priorities to learn.

```sql
WITH skills_demand AS (
    SELECT 
        sd.skill_id,
        sd.skills,
        COUNT(sj.job_id) AS demand_count
    FROM job_postings_fact j
    INNER JOIN skills_job_dim sj 
        ON j.job_id = sj.job_id
    INNER JOIN skills_dim sd 
        ON sj.skill_id = sd.skill_id
    WHERE 
        (j.job_location LIKE '%Netherlands%' OR
         j.job_location LIKE '%Germany%' OR
         j.job_location LIKE '%Belgium%') AND
        j.salary_year_avg IS NOT NULL 
    GROUP BY
        sd.skill_id, sd.skills
), average_salary AS (
    SELECT 
        sj.skill_id,
        ROUND(AVG(j.salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact j
    INNER JOIN skills_job_dim sj 
        ON j.job_id = sj.job_id
    INNER JOIN skills_dim sd 
        ON sj.skill_id = sd.skill_id
    WHERE 
        j.job_title_short LIKE '%Data%' AND 
        j.job_title_short LIKE '%Analyst%' AND 
        (j.job_location LIKE '%Netherlands%' OR
         j.job_location LIKE '%Germany%' OR
         j.job_location LIKE '%Belgium%') AND
        j.salary_year_avg IS NOT NULL 
    GROUP BY
        sj.skill_id
)

SELECT 
    sd.skill_id,
    sd.skills,
    sd.demand_count,
    a.avg_salary
FROM
    skills_demand sd
INNER JOIN 
    average_salary a ON sd.skill_id = a.skill_id
WHERE
    sd.demand_count > 10
ORDER BY
    a.avg_salary DESC,
    sd.demand_count DESC
LIMIT 20;
```
| Skill ID | Skill     | Demand Count | Avg Salary (€/year) |
| -------: | --------- | -----------: | ------------------: |
|        2 | nosql     |           24 |             167,081 |
|       98 | kafka     |           29 |             166,420 |
|      212 | terraform |           20 |             166,420 |
|      215 | flow      |           11 |             151,239 |
|      216 | github    |           11 |             150,896 |
|       81 | gcp       |           27 |             146,239 |
|       97 | hadoop    |           17 |             144,243 |
|       78 | redshift  |           22 |             142,910 |
|       77 | bigquery  |           17 |             140,499 |
|       80 | snowflake |           26 |             138,088 |
|       92 | spark     |          101 |             134,029 |
|        3 | scala     |           19 |             126,500 |
|       74 | azure     |           75 |             123,144 |
|       76 | aws       |           84 |             121,441 |
|       96 | airflow   |           34 |             111,202 |
|       95 | pyspark   |           18 |             111,184 |
|       93 | pandas    |           18 |             108,413 |
|      210 | git       |           35 |             108,088 |
|       14 | c#        |           11 |             107,769 |
|      182 | tableau   |           41 |             102,837 |


Key Insights:

- The most “optimal” skills skew heavily toward modern data engineering & cloud:
- NoSQL, Kafka, Terraform, GCP, Hadoop, Redshift, BigQuery, Snowflake, Spark all combine high salaries with non-trivial demand.
- Spark, Azure, and AWS stand out as skills with both high demand and strong pay, making them excellent long-term bets.
- Tools like Git/GitHub, Airflow, Pandas and languages like Scala, C# also score well, confirming that data analysts benefit from strong software engineering fundamentals.
- Tableau appears as a visualization tool with both solid demand and a six-figure average salary, reinforcing the value of strong communication and dashboarding skills.

pie title Top "Optimal" Skills by Avg Salary (Sample)
  "NoSQL" : 167081
  "Kafka" : 166420
  "Terraform" : 166420
  "Flow" : 151239
  "GitHub" : 150896
  "GCP" : 146239
  "Hadoop" : 144243
  "Redshift" : 142910

# What I learned?

Throughout this project, I’ve seriously leveled up my SQL + analysis game:

🧩 Complex Query Crafting: Chained multiple tables together with INNER and LEFT JOINs, used CTEs to structure logic, and wrote reusable queries to answer real job-market questions.

📊 Data Aggregation Power: Put GROUP BY, COUNT(), and AVG() to work to measure skill demand and salaries across countries, turning raw job postings into clear insights.

🧠 Analytical Problem-Solving: Translated vague questions like “What should I learn next?” into precise SQL queries, connecting business questions to data-driven answers.

🌍 Market Insight Building: Compared roles and skills across the Netherlands, Belgium, and Germany, learning how location, salary, and required skills all fit together in the data analyst job market.

# Closing thoughts

This project sharpened my SQL skills and gave me practical insight into the data analyst job market in the Netherlands, Belgium, and Germany. The analysis helps turn a messy landscape of job postings into clear guidance on which roles to target and which skills to prioritize.

By focusing on high-demand, high-salary skills, aspiring data analysts can position themselves more competitively and make smarter decisions about what to learn next. Overall, this exploration reinforced how important it is to keep learning, stay curious, and regularly revisit market data to stay aligned with the evolving needs of the data analytics field.