# HR Analytics Case Study by Oliver Roberts

---

### Project Overview

The aim of this case study is to determine possible reasons for employee dissatisfaction and attrition at the fictional French company Salifort Motors. We'll define business tasks, perform exploratory analysis, create accompanying charts followed by a dashboard, and produce recommendations with an aim to improve employee satisfaction at the company, and reduce attrition.

---

### Defining the business tasks

**Q1** : How many employees are working more than the standard 160 hours per month, and what is their job satisfaction level compared to workers who come in under that amount? 

Supporting [data](https://www.dol.gov/agencies/whd/fact-sheets/22-flsa-hours-worked#:~:text=The%20Act%20requires%20that%20employees,pay%20for%20the%20overtime%20hours.) from the US Department of Labor

**Q2** : Is there a relationship between the number of projects employees are assigned, and overall job dissatisfaction? Is there an ideal amount of projects, where employees show consistent satisfaction? 

**Q3** : Is there a relationship between time spent at company and job satisfaction?

**Q4** : How many employees experienced work accidents and left the company? How many stayed?

**Q5** : How does a promotion affect employee retention?

**Q6** : Are there particular departments that have a higher rate of attrition?

**Q7** : Which departments show the highest performance review score? Which departments show the lowest?

**Q8** : Are employees working more than the standard 160 hours per month scoring lower on their performance reviews than employees working the standard?

**Q9** : Are employees who score a high performance review getting rightfully promoted?

**Q10** : Are employees who stay at the company for 5 years or more, more satisfied overall than employees who stay at the company for less than 5 years?

**Q11** : Are employees that are working above the recommended amount of hours assigned too many projects?

**Q12** : How many employees left and what percentage of the total staff is it?

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
  ROW_NUMBER() OVER (PARTITION BY satisfaction_level, last_evaluation, number_project, average_monthly_hours, time_spend_company, work_accident, left_company,    promotion_last_5years, department, salary
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

Business Task Q1: How many employees are working more than the standard 160 hours per month, and what is their job satisfaction level compared to workers who come in under that amount? 

Firstly calculating the amount of employees working over 160 hours per month

```sql
SELECT COUNT(*) AS over_160_hours
  FROM dbo.HR_capstone_dataset
  WHERE average_monthly_hours > 160
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/26e49c0b-11a6-46b3-94b1-b83f6dd49eea)

We would then need to create bins for satisfaction_level so that we can group the employees and explore the data

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

To get an overview of how the employees satisfaction levels are distributed

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

---

Charts to view the data in a more pallatable means

---

A Pyramid Chart

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/1d59e3dd-508e-41fb-80d4-b25fa2245ca8)

A Scatter Plot

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/0b53d511-d8ad-4b08-b868-0679bd3b12fe)

---

For Business Task Q2 : Is there a relationship between the number of projects employees are assigned, and overall job dissatisfaction? Is there an ideal amount of projects, where employees show consistent satisfaction?

To observe the job satisfaction level of employees by the number of projects they are working on

```sql
SELECT AVG(satisfaction_level) AS avg_satisfaction_level, number_project
FROM dbo.HR_capstone_dataset_copy
GROUP BY number_project
ORDER BY avg_satisfaction_level ASC
```

![image](https://github.com/robertsoli/HR_Analysis/assets/156069037/f5b0fb5d-9835-457e-92d4-97715426589e)






