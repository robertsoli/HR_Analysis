# HR Analytics Case Study by Oliver Roberts

---

### Project Overview

The aim of this case study is to determine possible reasons for employee dissatisfaction and attrition at the fictional French company Salifort Motors. We'll define business tasks, perform exploratory analysis, create accompanying charts followed by a dashboard, and produce recommendations with an aim to improve employee satisfaction at the company, and reduce attrition.

---

### Defining the business tasks

**Q1** : 

**Q2** : 

**Q3** : 

**Q4** :

**Q5** :

**Q6** :

**Q7** :

**Q8** :

**Q9** :

**Q10** :


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


