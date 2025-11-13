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

# What I learned?

Throughout this project, I’ve seriously leveled up my SQL + analysis game:

🧩 Complex Query Crafting: Chained multiple tables together with INNER and LEFT JOINs, used CTEs to structure logic, and wrote reusable queries to answer real job-market questions.

📊 Data Aggregation Power: Put GROUP BY, COUNT(), and AVG() to work to measure skill demand and salaries across countries, turning raw job postings into clear insights.

🧠 Analytical Problem-Solving: Translated vague questions like “What should I learn next?” into precise SQL queries, connecting business questions to data-driven answers.

🌍 Market Insight Building: Compared roles and skills across the Netherlands, Belgium, and Germany, learning how location, salary, and required skills all fit together in the data analyst job market.

# Closing thoughts

This project sharpened my SQL skills and gave me practical insight into the data analyst job market in the Netherlands, Belgium, and Germany. The analysis helps turn a messy landscape of job postings into clear guidance on which roles to target and which skills to prioritize.

By focusing on high-demand, high-salary skills, aspiring data analysts can position themselves more competitively and make smarter decisions about what to learn next. Overall, this exploration reinforced how important it is to keep learning, stay curious, and regularly revisit market data to stay aligned with the evolving needs of the data analytics field.