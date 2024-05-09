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

### Data Cleaning and EDA 

- Data formats were corrected on import to MS Sql Server Management Studio

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

