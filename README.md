# Introduction
Gaining insights into the data job market! This project focuses on top-paying jobs, in-demand skills,
and where high demand meets high salary in data analytics.

Check out the SQL queries: [project_sql folder](/project_sql/)

# Background
Desiring to gain more experience in SQL, this project not only showcases queries but enhances my portfolio with
exposure to Postgres SQL, GitHub and Visual Studio

### Questions I answered with my queries:

1. What are the top paying data analyst jobs?
2. What skills are required for these top paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools I Used
The key tools I used for this project:

- **SQL:** Core tool for my analysis. Allowing me to query the database and uncover critical insights.

- **PostgreSQL:** The database management system I used to to handle the job posting data.

- **Visual Studio Code:** IDE used to manage the database and execute SQL queries.

- **Git and GitHub:** Used for version control and to share my SQL scripts, analysis, and project tracking.

# The Analysis
Each query in this project aimed at investigating specific aspects of the data analyst job market. Here's a breakdown of how each question was approached.

### 1. Top Paying Data Analyst Jobs
To identify the highest-paying roles, I filtered the results to Data Analyst positons by yearly average salary and location, 
focusing on remote jobs. This query highlights the highest paying roles based on the criteria. 

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    ROUND(salary_year_avg, 0) AS salary_year_avg,           -- rounding salary to whole number
    job_posted_date,
    name as company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id       -- left join to get company name
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 
    10
```

Here's the breakdown of the top paying data analyst jobs:
- **High-paying remote data roles:** All 10 jobs are full-time and remote, averaging $262K/year, with salaries ranging from $184K–$650K.

- **Leadership-heavy listings:** Most positions are Director or Principal level, indicating a focus on senior analytics roles.

- **Outlier detected:** The Mantys “Data Analyst” role at $650K is an extreme outlier and likely skews the average.

![Top Paying Roles](assets\1_top_paying_roles.png)
*Bar graph visualizing the top 10 highest salaries for data analyst roles. ChatGPT generated this bar graph from my SQL query results*

### 2. Skills Required for the Top Paying Data Analyst Jobs
To find the necessary skills for these high paying roles, I created a CTE, using the query from the first question. Inside the CTE, I joined the two company ID columns of the job_postings_fact table and the company_dim table. Outside the nested query I created two inner joins from the skills_job_dim and skill_dim tables. This query highlights the skills sought for these roles. 

```sql
WITH top_paying_jobs AS (               -- CTE to get top 10 paying Data Analyst jobs
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name as company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id          -- Join to get company names
    WHERE
        job_title_short = 'Data Analyst' AND                -- Filter for Data Analyst roles, remote locations
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT
        10
)

SELECT
    top_paying_jobs.*,
    skills
FROM
    top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id         -- Join to get skills for these jobs
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY 
    salary_year_avg DESC
```

Here's a summary of the skills needed for the top paying data analyst roles:
- **High-paying leadership roles dominate:** The top-paying positions, such as the Associate Director at AT&T ($255K), are leadership-heavy, indicating senior-level responsibilities and a broad set of required skills.

- **Specialized or cloud/analytics tools drive peaks:** Skills like Databricks, PySpark, Jupyter, and PowerPoint are associated with the very highest-paying roles, highlighting the premium placed on specialized technical expertise

- **Core analytics skills remain valuable:** SQL, Python, and Tableau consistently appear across multiple roles and companies, maintaining strong average salaries ($222K–$218K) even outside the top leadership roles

![Top Paying Skills](assets\top_paying_skills_graph.png)
*Bar graph visualizing the skills required for top paying data analyst roles. ChatGPT generated this bar graph from my SQL query results*


### 3. Skills Most in Demand for Data Analysts
To find the most in demand skills, I filtered the roles to Data Analyst and locations that are work from home. Then, I combined two tables on two different columns to get the skill names and jobs associated with each skill. This query highlights the most demanded skills in the field.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count                -- counting the number of jobs associated with the skill, renamed to demand count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id       -- Joining job postings with skills associated with each job
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id              -- Joining to get skill names
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = True
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5
```
Here's the summary for the most in demand skills for Data Analysts:
- **SQL dominates demand:** SQL is the most sought-after skill with 7,291 job listings, showing it remains the foundation of analytics and data roles.

- **Visualization tools show strong demand:** Tableau (3,745) and Power BI (2,609) indicate that companies are actively seeking professionals who can turn data into actionable insights through dashboards and reporting

## 📊 Top 5 Most In-Demand Skills for Data Analysts

| Rank | Skill     | Demand Count |
|------|------------|---------------|
| 1    | SQL        | 7,291         |
| 2    | Excel      | 4,611         |
| 3    | Python     | 4,330         |
| 4    | Tableau    | 3,745         |
| 5    | Power BI   | 2,609         |


### 4. Skills associated with higher salaries.
To find the skills associated with the highest salaries, I used the query from question 3, and filtered the results to Data Analyst roles, salary average is unique, and where the job location is work from home. The query highlights the skills correlating to higher salaries.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id       -- Join to get skills associated with each job
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id                -- Join to get skill names
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = True
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT
    25
```

Breakdown of the highest paid skills in the field:
- **Shift toward data engineering & scalable analytics:** Tools like PySpark, Databricks, Airflow, and GCP dominate — showing that analysts who can manage big data pipelines and cloud systems earn the most.

- **Machine learning & automation skills drive premiums:** High salaries are linked to ML and AI tools (DataRobot, Watson, scikit-learn, pandas), reflecting demand for analysts who move beyond reporting to predictive insights.

- **Full-stack, cross-functional analysts are most valuable:** Combining DevOps (GitLab, Jenkins), programming, and communication tools (Notion, MicroStrategy) signals that analysts who bridge data, engineering, and business deliver — and earn — the most.

## 💰 Top 25 Highest-Paying Skills for Data Professionals

| Rank | Skill           | Average Salary ($) |
|------|------------------|--------------------|
| 1    | PySpark          | 208,172            |
| 2    | Bitbucket        | 189,155            |
| 3    | Couchbase        | 160,515            |
| 4    | Watson           | 160,515            |
| 5    | DataRobot        | 155,486            |
| 6    | GitLab           | 154,500            |
| 7    | Swift            | 153,750            |
| 8    | Jupyter          | 152,777            |
| 9    | Pandas           | 151,821            |
| 10   | Elasticsearch    | 145,000            |
| 11   | Golang           | 145,000            |
| 12   | NumPy            | 143,513            |
| 13   | Databricks       | 141,907            |
| 14   | Linux            | 136,508            |
| 15   | Kubernetes       | 132,500            |
| 16   | Atlassian        | 131,162            |
| 17   | Twilio           | 127,000            |
| 18   | Airflow          | 126,103            |
| 19   | Scikit-learn     | 125,781            |
| 20   | Jenkins          | 125,436            |
| 21   | Notion           | 125,000            |
| 22   | Scala            | 124,903            |
| 23   | PostgreSQL       | 123,879            |
| 24   | GCP              | 122,500            |
| 25   | MicroStrategy    | 121,619            |



### 5. The Most Optimal(High-Demand and High-Pay) Skills to Learn
To find the most optimal skills in the field, I used inner join for the job and skill tables on job ID, and the skills and skill_dim tables on skill ID, then filtered my results to Data Analyst roles, jobs listed with an average salary, and locations where you can work from home. This query highlights the most optimal skills to learn.

```sql
SELECT
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id   -- join fact to job-skill bridge
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id          -- join bridge to skills dimension
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL              -- exclude jobs without salary data
    AND job_work_from_home = True
GROUP BY
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10             -- filter for skills with more than 10 job postings
ORDER BY
    demand_count DESC,
    avg_salary DESC
LIMIT 25
```

Breakdown of the most optimal skills to learn:
- **Data & Analytics Dominate:** Skills like SQL, Python, Excel, Tableau, Power BI, and R have the highest demand counts (ranging from 100–400+), showing that data analysis, visualization, and manipulation remain core to high-paying roles across industries.

- **Cloud & Engineering Skills Pay Premiums:** Technologies such as AWS, Azure, Go, Hadoop, Snowflake, and Oracle offer some of the highest average salaries ($105K–$115K), even with lower demand, reflecting their technical depth and enterprise value.
    
- **Productivity Tools Still Matter:** Traditional tools like Excel, PowerPoint, Word, and Sheets continue to appear frequently, confirming that hybrid analytical–communication skills are valuable complements to technical expertise.

## 💼 Top 25 High-Pay, High-Demand Skills

| Rank | Skill       | Demand Count | Average Salary ($) |
|------|--------------|---------------|--------------------|
| 1    | SQL          | 398           | 97,237             |
| 2    | Excel        | 256           | 87,288             |
| 3    | Python       | 236           | 101,397            |
| 4    | Tableau      | 230           | 99,288             |
| 5    | R            | 148           | 100,499            |
| 6    | Power BI     | 110           | 97,431             |
| 7    | SAS          | 63            | 98,902             |
| 8    | PowerPoint   | 58            | 88,701             |
| 9    | Looker       | 49            | 103,795            |
| 10   | Word         | 48            | 82,576             |
| 11   | Snowflake    | 37            | 112,948            |
| 12   | Oracle       | 37            | 104,534            |
| 13   | SQL Server   | 35            | 97,786             |
| 14   | Azure        | 34            | 111,225            |
| 15   | AWS          | 32            | 108,317            |
| 16   | Sheets       | 32            | 86,088             |
| 17   | Flow         | 28            | 97,200             |
| 18   | Go           | 27            | 115,320            |
| 19   | SPSS         | 24            | 92,170             |
| 20   | VBA          | 24            | 88,783             |
| 21   | Hadoop       | 22            | 113,193            |
| 22   | Jira         | 20            | 104,918            |
| 23   | JavaScript   | 20            | 97,587             |
| 24   | SharePoint   | 18            | 81,634             |



# What I Learned
Throughout the project, I elevated my SQL skills and learned to put them into actions:

- **Complex Query Crafting:** Designed and optimized complex queries using joins, subqueries, and aggregations to support data analysis and reporting.

- **Analytical Reporting:** Used the results from complex queries to generate detailed analytical reports, summarizing key metrics, uncovering trends, and providing actionable insights for decision-making.

- **Data Aggregation:** Gained experience merging tables to consolidate data and generate actionable insights.

# Conclusions

### Insights:
1. **Top Paying Data Analyst Jobs**: The results of top paying jobs are roles are Director or Principal level roles, which shows there's a demand for Senior Level Analytical experience in the market. Additionally, the Mantys Data Analyst Role salary is an outlier to the rest of the results, causing the average of salary to be skewed.

2. **Skills for Top Paying Jobs**: Skills like Databricks, PySpark, Jupyter, and PowerPoint are associated with the very highest-paying roles, highlighting the premium placed on specialized technical expertise. In addition, SQL, Python, and Tableau consistently appear across multiple roles and companies, maintaining strong average salaries. Lastly, the top paying positions are leadership-heavy, indicating senior-level responsibilities and a broad set of required skills.

3. **Most In Demand Skills**: SQL is the most sought-after skill with 7,291 job listings, showing it remains the foundation of analytics and data roles. Additionally, roles requiring experience in Tableau and Power BI indicate that companies are actively seeking professionals who can turn data into actionable insights through dashboards and reporting. 

4. **Skills with Higher Salaries**: Tools like PySpark, Databricks, Airflow, and GCP dominate — showing that analysts who can manage big data pipelines and cloud systems earn the most. Additionally, combining DevOps (GitLab, Jenkins), programming, and communication tools (Notion, MicroStrategy) signals that analysts who bridge data, engineering, and business deliver — and earn — the most. Demand for analysts who move beyond reporting to predictive insights are linked to higher salaries as well with AI and Machine Learning experience. 

5. **Optimal Skills for Job Market Value**: Skills like SQL, Python, Excel, Tableau, Power BI, and R have the highest demand counts, showing that data analysis, visualization, and manipulation remain core to high-paying roles across industries. Traditional tools like Excel, PowerPoint, Word, and Sheets continue to appear frequently, confirming that hybrid analytical–communication skills are valuable complements to technical expertise.

### Closing Thoughts
This project enhanced my SQL skills and simulated a SQL project that utilized a database management system, a DB editor, and exposure to Git and GitHub. Not only does this project elevated my analysis skills, I was able to create complex queries I have not tried before. The findings of my analysis were able to give insight into some of the skills needed to land some data analyst roles. While the data is 2 years old, the experience needed for job postings today is still relevant. This exploration highlights the importance continuous learning as the job market continous to shift towards utilizing new technological advancements to perform in these roles. 