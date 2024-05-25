# HR Analytics Case Study
## by Oliver Roberts

---

### Project Overview

The aim of this case study is to determine possible reasons for employee dissatisfaction and attrition at the fictional French company Salifort Motors. We'll define business tasks, perform exploratory analysis, create accompanying charts followed by a dashboard, and produce recommendations with an aim to improve employee satisfaction at the company, and reduce attrition.

---

### Defining the business tasks

**Q1** : How many employees are working more than the standard 160 hours per month, and what is their job satisfaction level compared to workers who come in under that amount? 

Supporting [data](https://www.dol.gov/agencies/whd/fact-sheets/22-flsa-hours-worked#:~:text=The%20Act%20requires%20that%20employees,pay%20for%20the%20overtime%20hours.) from the US Department of Labor

**Q2** : Is there a relationship between the number of projects employees are assigned, and overall job dissatisfaction? Is there an ideal amount of projects, where employees show consistent satisfaction? 

**Q3** : How does time spent at the company affect overall job satisfaction?

**Q4** : How many employees experienced work accidents and left the company? How many stayed?

**Q5** : How does a promotion affect employee retention?

**Q6** : Are there particular departments that have a higher rate of attrition?

**Q7** : Which departments show the highest performance review score for active employees? Which departments show the lowest?

**Q8** : How do monthly working hours influence performance scores?

**Q9** : Does project count contribute to employee attrition?

**Q10** : Are certain departments assigned more projects than others?

**Q11** : How do evaluation scores fluctuate with salary brackets? 

**Q12** : How are salary brackets distributed per department? 

---

### Data Sources

- This publicly available data was obtained on Kaggle [here](https://www.kaggle.com/datasets/raminhuseyn/hr-analytics-data-set)

- The table used in this case study is HR_capstone_dataset.csv

- [Guidelines](https://www.dol.gov/agencies/whd/fact-sheets/22-flsa-hours-worked#:~:text=The%20Act%20requires%20that%20employees,pay%20for%20the%20overtime%20hours.) from the US Department of Labor on recommended working week hours.

---

### Tools used in analysis

- Microsoft SQL Server Management Studio 20, using T-SQL for cleaning tasks, exploratory data analysis and manipulation
- Tableau for related charts and a dashboard

---

### Data Cleaning 

- Create a copy of the dataset that will be cleaned and manipulated, so that the raw data is still in tact

- Data formats were corrected on import to MS Sql Server Management Studio

- Checking for null values

- Finding, inspecting and removing duplicate entries

---

### Data Manipulation

- Renaming of variables

- 

- Start by cleaning the data, looking at the business tasks as which new variables need to be made may only become apparent in motion

---

#### Renaming variables to follow naming conventions

```sql

EXEC 
sp_rename 'dbo.HR_capstone_dataset.left', 'left_company', 'COLUMN';
GO

EXEC 
sp_rename 'dbo.HR_capstone_dataset.Work_accident', 'work_accident', 'COLUMN';
GO

EXEC 
sp_rename 'dbo.HR_capstone_dataset.Department', 'department', 'COLUMN';
GO

EXEC 
sp_rename 'dbo.HR_capstone_dataset.average_montly_hours', 'average_monthly_hours', 'COLUMN';
GO

```

---

#### Checking for null values

```sql
SELECT *
FROM dbo.HR_capstone_dataset
WHERE satisfaction_level IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE last_evaluation IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE number_project IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE average_monthly_hours IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE time_spend_company IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE work_accident IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE left_company IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE promotion_last_5years IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE department IS NULL
GO

SELECT *
FROM dbo.HR_capstone_dataset
WHERE salary IS NULL
GO
```

---

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/f2b4f39e-68ae-4e21-bfe6-ad07d6fc68ca)

---

#### Checking for duplicate entries

```sql
SELECT satisfaction_level, last_evaluation, number_project, average_monthly_hours, time_spend_company, work_accident, left_company, promotion_last_5years, department, salary, COUNT(*) AS duplicates
FROM HR_capstone_dataset
GROUP BY satisfaction_level, last_evaluation, number_project, average_monthly_hours, time_spend_company, work_accident, left_company, promotion_last_5years, department, salary
HAVING COUNT(*) > 1
ORDER BY duplicates DESC;
```

---

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/5e08f463-9e23-428b-8382-f0dc7ba6848a)

---

#### Removing the duplicate entries

```sql
WITH DuplicateCTE AS
(
SELECT *,
  ROW_NUMBER() OVER (PARTITION BY satisfaction_level, last_evaluation, number_project, average_monthly_hours, time_spend_company, work_accident, left_company,    
                                  promotion_last_5years, department, salary
ORDER BY (SELECT NULL)) AS RowNum
FROM HR_capstone_dataset
)
DELETE FROM DuplicateCTE WHERE RowNum > 1;
```

---

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/a3047524-2a71-43f8-a144-6c616cf478ff)

> [!NOTE]
> Ran the query searching for duplicates again, none found. Reason for the difference in rows affected is that there were multiple duplicates of the same rows, in some cases as many as 6. 

---

### Exploratory Data Analysis

---

**Business Task Q1**

How many employees are working more than the standard 160 hours per month, and what is their job satisfaction level compared to workers who come in under that amount? 

Firstly calculating the amount of employees working both over and under 160 hours per month :

```sql
SELECT SUM(CASE WHEN average_monthly_hours > 160 THEN 1 ELSE 0 END) AS over_160_hours,
       SUM(CASE WHEN average_monthly_hours <= 160 THEN 1 ELSE 0 END) AS under_160_hours
FROM dbo.HR_capstone_dataset_copy
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/8ca93da4-8d7b-45fa-ba17-7b49632ad981)

We would then need to create bins for satisfaction_level so that we can group the employees and explore the data :

```sql
ALTER TABLE dbo.HR_capstone_dataset_copy ADD satisfaction_group varchar(50);

UPDATE dbo.HR_capstone_dataset_copy  SET satisfaction_group =
(CASE WHEN satisfaction_level <=0.1 THEN '0-10%'
	  WHEN satisfaction_level > 0.1 AND satisfaction_level <=0.2 THEN '10-20%'
    WHEN satisfaction_level > 0.2 AND satisfaction_level <=0.3 THEN '20-30%'
	  WHEN satisfaction_level > 0.3 AND satisfaction_level <=0.4 THEN '30-40%'
	  WHEN satisfaction_level > 0.4 AND satisfaction_level <=0.5 THEN '40-50%'
	  WHEN satisfaction_level > 0.5 AND satisfaction_level <=0.6 THEN '50-60%'
	  WHEN satisfaction_level > 0.6 AND satisfaction_level <=0.7 THEN '60-70%'
    WHEN satisfaction_level > 0.7 AND satisfaction_level <=0.8 THEN '70-80%'
	  WHEN satisfaction_level > 0.8 AND satisfaction_level <=0.9 THEN '80-90%'
	  WHEN satisfaction_level > 0.9 AND satisfaction_level <=1 THEN '90-100%'
END);
```

To get an overview of how the employees satisfaction levels are distributed :

```sql
SELECT satisfaction_group, COUNT(*) AS number_of_employees
  FROM [HR Analytics].[dbo].[HR_capstone_dataset_copy]
  GROUP BY satisfaction_group
  ORDER BY satisfaction_group ASC
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/79bf71ea-1f68-4ce0-b05d-b08cf81bc343)

To observe how employees satisfaction levels are distributed when employees are working over or under the recommended 160 monthly hours

```sql
SELECT
    satisfaction_group,
    SUM(CASE WHEN average_monthly_hours <= 160 THEN 1 ELSE 0 END) AS less_than_160_hours,
    SUM(CASE WHEN average_monthly_hours > 160 THEN 1 ELSE 0 END) AS more_than_160_hours
FROM dbo.HR_capstone_dataset_copy
GROUP BY satisfaction_group
ORDER BY satisfaction_group ASC;
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/46e2b601-585b-4f74-bc2f-741e56f5d490)

Charts to zoom out and view the data in a more pallatable means :

A Pyramid Chart

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/068936da-b5c1-4330-a411-5e8be620dee9)

A Scatter Plot

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/0b53d511-d8ad-4b08-b868-0679bd3b12fe)

#### Observations :

Datapoints are distributed somewhat randomly besides two obvious clusters: 

- Employees working between 120 and 160 hours show a low satisfaction level, suggesting that being underworked is also a contributor to lower satisfaction levels.
  
- The other cluster is where employees are working between 240 and 310 hours, which is definitely a result of being overworked.

---

**Business Task Q2** 

Is there a relationship between the number of projects employees are assigned, and overall job dissatisfaction? Is there an ideal amount of projects, where employees show consistent satisfaction?

To observe the job satisfaction level of employees by the number of projects they are working on :

```sql
SELECT AVG(satisfaction_level) AS avg_satisfaction_level, number_project
FROM dbo.HR_capstone_dataset_copy
GROUP BY number_project
ORDER BY avg_satisfaction_level ASC
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/f5b0fb5d-9835-457e-92d4-97715426589e)

A Bar Chart showing Average Satisfaction Level by Number of Projects

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/90b1945c-be0a-493d-b022-312bcfbdc8e5)



#### Observations :

- We can clearly see that when employees are tasked with 7 projects monthly, that there is a massive drop in satisfaction level, and that employees tasked with 
  6 projects monthly also show a very low satisfaction score. 

- Employees tasked with 3, 4 and 5 projects record the highest level of satisfaction at an average of 0.69

---

**Business Task Q3**

How does time spent at the company affect overall job satisfaction? 

First lets have a look into the average satisfaction levels by tenure :

```sql
SELECT time_spend_company, AVG(satisfaction_level) AS avg_satisfaction
FROM dbo.HR_capstone_dataset_copy
GROUP BY time_spend_company
ORDER BY time_spend_company
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/880ccce8-ba14-4cea-9fc5-22750b0575cd)



To drill down on the low satisfaction level at 4 years tenure, lets look at the promotion ratio per tenure:

```sql
SELECT time_spend_company, 
	SUM(promotion_last_5years) AS promotion_count, 
	COUNT(promotion_last_5years) AS employees_in_department,
	CAST((SUM(promotion_last_5years) * 100.0 / COUNT(promotion_last_5years)) AS DECIMAL(10,2)) AS promotion_percentage
FROM dbo.HR_capstone_dataset_copy
GROUP BY time_spend_company
ORDER BY time_spend_company
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/589e6fed-2f1d-4461-8f59-6d368ddc1630)

A Bar Chart to show the data in a more visual sense

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/94a77a68-267a-46b0-8ec9-4b331d248796)

#### Observations :

- The lowest average satisfaction level is observed in employees who have been at the company for 4 years.

  - This may be due to employees having worked at the company for 4 years see only a promotion ratio of 1.10%, with average satisfaction climbing from 5 years 
    up and promotion ratio climbing from 6 years and up. 

- The highest average satisfaction level is observed in employees who have been at the company for 2 years. 

  - This may be due to the fact that new employees tend to be excited about joining a company, and have not been exposed to a lengthy term of being overworked 
    in the same business.

- Employees working at the company for 6 years plus show a consistent upward trend in satisfaction level.

---

**Business task Q4**

How many employees experienced work accidents and left the company? How many stayed?

```sql
SELECT 
COUNT(*) AS number_of_employees, 
work_accident, 
left_company
FROM dbo.HR_capstone_dataset_copy
WHERE work_accident = 1
GROUP BY work_accident, left_company
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/e1f6459a-9493-499c-a38a-17c79bea62c6)

A calculation to determine the ratio of employees who experienced a workplace accident and stayed:

```sql
SELECT
    SUM(CASE WHEN work_accident = 1 AND left_company = 0 THEN 1 ELSE 0 END) AS accident_stayed,
    SUM(CASE WHEN work_accident = 1 AND left_company = 1 THEN 1 ELSE 0 END) AS accident_left,
    (SUM(CASE WHEN work_accident = 1 AND left_company = 1 THEN 1 ELSE 0 END) * 100) / NULLIF(SUM(CASE WHEN work_accident = 1 AND left_company = 0 THEN 1 ELSE 0 END), 0) AS effect_of_accident_ratio
FROM dbo.HR_capstone_dataset_copy;
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/33fe8ab7-940b-4199-aaf5-fb54e7f13ca6)

A calculation to determine the ratio of employees who experienced a workplace accident during their tenure:

```sql
SELECT
COUNT(*) AS count_of_employees,
SUM(CASE WHEN work_accident = 1 THEN 1 ELSE 0 END) AS work_accidents,
(SUM(CASE WHEN work_accident = 1 THEN 1 ELSE 0 END)* 100) / NULLIF(COUNT(*),0) AS percentage_of_accidents
FROM dbo.HR_capstone_dataset_copy
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/a95a654e-fa8d-4cce-bed5-f78297804397)



#### Observations

- Based on the very low percentage of 6% of employees leaving the company after a workplace accident, it surprisingly seems like workplace accidents are not a major factor in employees leaving the company.

- With work place accidents coming in at a percentage of 15.4% of all employees, which is roughly one in 6 people, this is no doubt a serious problem.

---

#### Business task Q5

**How does a promotion affect employee retention?**

Looking at the number of employees who have left the company without receiving a promotion, and the amount of employees who left after receiving a promotion would be of use in this scenario: 

```sql
SELECT 
    promotion_last_5years,
    SUM(CASE WHEN left_company = 1 THEN 1 ELSE 0 END) AS left_company,
    SUM(promotion_last_5years) AS total_promotions
FROM 
    dbo.HR_capstone_dataset_copy
GROUP BY 
    promotion_last_5years;
```  

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/2d6a0d01-5388-4d12-8f5b-78832c25a741)

To dig a bit deeper into the reasons behind the 8 employees who left despite receiving a promotion, we could draw up a table of those 8 records: 

```sql
SELECT *
FROM dbo.HR_capstone_dataset_copy
WHERE left_company = 1 AND promotion_last_5years = 1
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/0e946a4e-bd8d-4a1b-a791-255086dbced5)

#### Observations

- Based on the result set, only 8 employees out of 203 left after receiving a promotion.

- 1983 employees left the company without a promotion, and although this could be for a plethora of other reasons, employee retention is definitely higher after receiving a promotion.

- Looking for trends between these 8 employees, we could possibly relate their departure to :

  - Low satisfaction level due to working on 2 or 6 projects which accounts for 6 records
  - Extremely high monthly hour count, as seen in 2 records
  - Low and medium salaries as seen in all the records

---

#### Business task Q6 

**Are there particular departments that have a higher rate of attrition?**

Here we break down the count of employees who left the department, the total number of active employees in the department, and the percentage of employees who left the company: 

```sql
SELECT department,
SUM(left_company) AS left_company,
SUM(CASE WHEN left_company = 0 THEN 1 ELSE 0 END) AS total_active_employees,
(SUM(CASE WHEN left_company = 1 THEN 1 ELSE 0 END) * 100) / NULLIF(SUM(CASE WHEN left_company = 0 THEN 1 ELSE 0 END), 0) AS percentage_left
FROM dbo.HR_capstone_dataset_copy
GROUP BY department
ORDER BY left_company DESC
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/495fe4ff-9784-4e5d-b5a4-be94bfdb7dcd)

And a horizontal bar chart to vizualise the data

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/af5ee1cb-5b15-4f5a-bbcd-5bb5170c8ac7)

#### Observations

 - Although the Sales department had the higher count of employees who left the company, this is due to it being the largest department overall. Percentage 
   wise, there is not a department with a significantly higher rate of attrition than others. Both RND and Management have the lowest rate of attrition, coming 
   in at 13%. 

---

#### Business task Q7

**Which departments show the highest performance review score for active employees? Which departments show the lowest?**

For this query, it would be useful to create bins of 10% each, and view the performance evaluation counts in those bins, by department:

```sql
SELECT 
	department,
	COUNT(last_evaluation) AS total_active_employees,
	SUM(CASE WHEN evaluation_group = '0-10%' THEN 1 ELSE 0 END) AS zero_to_ten,
	SUM(CASE WHEN evaluation_group = '10-20%' THEN 1 ELSE 0 END) AS ten_to_twenty,
	SUM(CASE WHEN evaluation_group = '20-30%' THEN 1 ELSE 0 END) AS twenty_to_thirty,
	SUM(CASE WHEN evaluation_group = '30-40%' THEN 1 ELSE 0 END) AS thirty_to_fourty,
	SUM(CASE WHEN evaluation_group = '40-50%' THEN 1 ELSE 0 END) AS fourty_to_fifty,
	SUM(CASE WHEN evaluation_group = '50-60%' THEN 1 ELSE 0 END) AS fifty_to_sixty,
	SUM(CASE WHEN evaluation_group = '60-70%' THEN 1 ELSE 0 END) AS sixty_to_seventy,
	SUM(CASE WHEN evaluation_group = '70-80%' THEN 1 ELSE 0 END) AS seventy_to_eighty,
	SUM(CASE WHEN evaluation_group = '80-90%' THEN 1 ELSE 0 END) AS eighty_to_ninety,
	SUM(CASE WHEN evaluation_group = '90-100%' THEN 1 ELSE 0 END) AS ninety_to_one_hundred
FROM dbo.HR_capstone_dataset_copy
WHERE left_company = 0
GROUP BY department
ORDER BY total_active_employees DESC;
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/caacc4c3-9f61-4916-966a-952b82c28dd9)

Based on the amount of data in the table, it would be more useful to view it as a a bar chart :

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/be54e415-7013-4777-87e5-5f44db0d4d0b)

#### Observations

- Although this chart feels cramped, it does make it easier to draw the following conclusions:
  - At 18%, HR comes in with the highest number of employees receiving a 90-100% performance review, as well as 55.5% of the department scoring 70% or more in evaluation score. 
  - At 3.1%, Product Management comes in with the highest number of employees receiving a 30-40% performance review, as well as 11.4% of the department scoring less than 50% in evaluation score.
 
---

#### Business task Q8

**How do monthly working hours influence performance scores?**

Based on the amount of data points, a Highlight Table would be best :

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/b2cd3cab-7d93-479a-be23-cb1ef7bba935)

#### Observations

Should higher working hours be a strong indicator of poor performance evaluation scores, we would expect to see a cluster of employees in the lower performance bins over the 160 hours per month mark, but that is not the case. Observations :

  - There is a cluster of employees with performance evaluations between 40 and 60%, when working between 130 to 150 hours per month.
  - There is a cluster of employees with performance evaluations between 80 and 100%, when working between 240 to 260 hours per month.

Overall, it seems that employees working more than 160 hours per month are getting higher evaluation scores, this may be due to the following reasons:

 - The company places a high value on employees who work more than the standard amount, thus the higher performance scores for employees working in excess of 160 hours per month.
 - Employees may be paid overtime, resulting in higher motivation to work longer hours monthly at a high level of performance.

---

#### Business task Q9 

**Does project count contribute to employee attrition?**

In order to get a view of the employees who left the company grouped by the number of projects they were working on : 

```sql
SELECT 
    number_project, 
    employees_left, 
    employees_stayed, 
    total_employees, 
    CAST(employees_left AS DECIMAL) / total_employees * 100 AS percentage_left
FROM (
    SELECT 
        number_project,
        SUM(CASE WHEN left_company = 1 THEN 1 ELSE 0 END) AS employees_left,
        SUM(CASE WHEN left_company = 0 THEN 1 ELSE 0 END) AS employees_stayed,
        COUNT(CASE WHEN left_company IN (0, 1) THEN 1 END) AS total_employees
    FROM 
        dbo.HR_capstone_dataset_copy
    GROUP BY 
        number_project
) AS subquery;
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/d78462f5-ce98-40cd-84ee-797d5dd1fafe)

Below is a custom Bar Chart showing the attrition rate by number of projects:

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/25063540-ddc1-421d-bb5e-4bdd4233f943)

Based on the data, the following observations can be made:

 - All employees who were assigned to 7 Projects left the company
 - 45% of employees who were assigned to 6 projects left the company
 - 54% of employees who were assigned to 2 projects left the company
 - When assigned 3, 4 or 5 projects attrition is significantly lower

---

#### Business task Q10

**Are certain departments assigned more projects than others?**

Lets create a table for the departments, their average number of projects, and a count of the number of projects per department: 

```sql
SELECT department,
AVG(CAST(number_project AS FLOAT)) AS average_projects,
SUM(CASE WHEN number_project = 7 THEN 1 ELSE 0 END) AS seven_projects,
SUM(CASE WHEN number_project = 6 THEN 1 ELSE 0 END) AS six_projects,
SUM(CASE WHEN number_project = 5 THEN 1 ELSE 0 END) AS five_projects,
SUM(CASE WHEN number_project = 4 THEN 1 ELSE 0 END) AS four_projects,
SUM(CASE WHEN number_project = 3 THEN 1 ELSE 0 END) AS three_projects,
SUM(CASE WHEN number_project = 2 THEN 1 ELSE 0 END) AS two_projects
FROM dbo.HR_capstone_dataset_copy
GROUP BY department
ORDER BY seven_projects DESC
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/357c6097-2633-4e82-b34a-7fabd0031ab6)

Based on the data, we can make the following observations: 

- There is no significant difference in the average number of projects being assigned for any department
- Based on the size of the sales department, the numbers are obviously higher for employees assigned a problematic number of projects, but in reality the ratio is almost exactly the same with all the other departments.

> [!NOTE]  
> There wont be a useful chart to display the data differently as we have the answer to our business task from the query results

---

#### Business task Q11 

**How do evaluation scores fluctuate with salary brackets?**

For the purpose of viewing these data, a visualization is the simplest way to look for any trends, so a Bar Chart gave the best visual cues:

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/12f0b5d4-02e7-4b62-bc52-a8295c46fe8a)

---

#### Business task Q12

**How are salary brackets distributed per department?**

It would be useful to determine the number of employees that fall into different salary brackets, and the percentage thereof : 

```sql
SELECT 
    department,
	total_employees,
    low_salary_count, 
    medium_salary_count, 
    high_salary_count, 
    CAST(low_salary_count AS DECIMAL) / total_employees * 100.0 AS low_percentage,
	CAST(medium_salary_count AS DECIMAL) / total_employees * 100 AS medium_percentage,
	CAST(high_salary_count AS DECIMAL) / total_employees * 100 AS high_percentage
FROM (
    SELECT 
        department,
		COUNT(*) AS total_employees,
SUM(CASE WHEN salary = 'low' THEN 1 ELSE 0 END) AS low_salary_count,
SUM(CASE WHEN salary = 'medium' THEN 1 ELSE 0 END) AS medium_salary_count,
SUM(CASE WHEN salary = 'high' THEN 1 ELSE 0 END) AS high_salary_count
    FROM 
        dbo.HR_capstone_dataset_copy
GROUP BY department
)
AS subquery;
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/875d82fd-e8b3-4bfb-a4bd-38b8e17680ab)

Based on the size differences in the number of employees in each department, the distribution is best viewed as a percentage in a bar chart form:

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/31933571-6ac8-4891-bfe6-2ad917e57afc)




