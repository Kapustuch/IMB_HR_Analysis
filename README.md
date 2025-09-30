# IMB HR Data Analysis

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Final Results & Business Recommendations](#final-results--business-recommendations)

## Project Overview

This project analyzes the **IBM HR Analytics Dataset** to uncover key workforce insights and identify the drivers of employee attrition.  
Employee turnover is one of the most critical challenges faced by organizations, leading to high replacement costs, productivity loss, and reduced morale.  

The goal of this project is to explore workforce patterns, test hypotheses using **statistical methods**, and provide actionable insights that HR managers can use to improve retention strategies.  

The analysis combines:
- **Exploratory Data Analysis (EDA):** Understanding attrition by age, job role, department, salary, tenure, and work-life balance.  
- **Statistical Testing:** Applying T-tests, Chi-square, and ANOVA to validate whether factors like compensation, overtime, and satisfaction significantly affect attrition.  
- **Business Insights:** Translating findings into clear recommendations for HR policies, such as improving onboarding, addressing pay equity, and managing workload.  

By linking statistical evidence with HR practices, this project provides a data-driven foundation for reducing attrition, improving employee satisfaction, and lowering organizational costs.

## Data Sources

The dataset used in this project is the **IBM HR Analytics Employee Attrition & Performance Dataset**.  
It contains **1,470 employee records** with **35 attributes** covering demographic details, job characteristics, compensation, satisfaction levels, and attrition status.  

### Dataset Highlights
- **Target Variable:** `Attrition` (Yes/No) — whether the employee left the company.  

- **Key Demographic Features:** Age, Gender, Marital Status, Education, Education Field.  

- **Job-Related Features:** Department, Job Role, Years at Company, Years in Current Role, Business Travel, Overtime.  

- **Compensation & Performance Features:** Monthly Income, Percent Salary Hike, Stock Option Level, Training Times Last Year, Performance Rating.  

- **Satisfaction & Work-Life Features:** Job Satisfaction, Environment Satisfaction, Work-Life Balance, Relationship Satisfaction.  

You can access the dataset and its files here: [IBM HR Analytics Employee Attrition & Performance Dataset on Kaggle](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset).

## Tools

- **Python** – core programming language for data analysis  
- **pandas** – data manipulation and cleaning  
- **numpy** – numerical computations and array operations  
- **matplotlib** – data visualization and plotting  
- **seaborn** – advanced statistical visualizations
- **scipy.stats** – statistical hypothesis testing (t-tests, ANOVA, chi-square, etc.)

## Exploratory Data Analysis

The EDA focused on understanding workforce demographics, job characteristics, compensation, and satisfaction levels, as well as how these factors relate to employee attrition.  
Key descriptive analyses included:  

- **Overall attrition rate:** 16.1% of employees left the company.  
- **Attrition by demographics:**  
  - Younger employees (18–29) showed the highest attrition rate (27.9%).  
  - Male attrition (17%) was slightly higher than female attrition (14.8%).  
- **Attrition by job role and department:**  
  - The Sales Department(20.6%) and Sales Representatives(39.8%) show the highest attrition among all departments and job roles.
  
- **Compensation patterns:**  
  - Strong inverse relationship between salary and attrition — 34% of employees earning under 2,500 left, compared to 9% earning 10,000–20,000.  
- **Tenure analysis:**  
  - Highest attrition occurred during the first 1–2 years of employment, then dropped sharply after 5 years.  
- **Work-life balance and travel:**  
  - Employees with poor work-life balance or frequent travel were significantly more likely to leave.  

These findings were supported with **visualizations** such as bar charts, line plots, and boxplots, which highlighted key workforce patterns and attrition risk factors.  

A detailed summary of these insights, along with statistical testing results and recommendations, can also be found in the full [PDF Report](HR_Analytics_Report.pdf).

## Data Analysis

### Example: Monthly Income vs Attrition
One key analysis was to explore whether salary is a significant factor in employee attrition.  
The code below demonstrates how salaries differ between employees who left and those who stayed, identifies which Monthly Income group has the highest attrition rate, and uses a two-sample t-test to determine whether this difference is statistically significant.

```python
df[df['Attrition']=='Yes']['MonthlyIncome'].mean().round(2)
```
`4787.09`

```python
df[df['Attrition']=='No']['MonthlyIncome'].mean().round(2)
```
`6832.74`

```python
plt.figure(figsize=(6,5))
sns.boxplot(x="Attrition", y="MonthlyIncome", data=df)

plt.title("Monthly Income by Attrition Status", fontsize = 12, weight = 'bold')
plt.xlabel('Attiriton', fontsize = 10)
plt.ylabel('Monthly Income', fontsize = 10)

plt.tight_layout()
```
<img width="590" height="490" alt="image" src="https://github.com/user-attachments/assets/0dae3d59-b310-4e90-b60b-abe9f26d3a46" />

```python
income_bins = [0, 2500, 5000, 10000, 20000]
income_labels = ['0-2500', '2500-5000', '5000-10000', '10000-20000']
df['MonthlyIncomeGroup'] = pd.cut(df['MonthlyIncome'], bins=income_bins, labels=income_labels)

all_income_groups = df.groupby('MonthlyIncomeGroup', observed=False).size().reset_index(name='Count')
attrition_income_groups = (
    df[df['Attrition'] == 'Yes']
    .groupby('MonthlyIncomeGroup', observed=False)
    .size()
    .reset_index(name='Attrition Count')
)

income_results = all_income_groups.merge(attrition_income_groups, on='MonthlyIncomeGroup')

income_results['% of Attrition'] = np.round(income_results['Attrition Count'] / income_results['Count'] * 100, 2)
```

```python
income_results
```
| MonthlyIncomeGroup | Count | Attrition Count | % of Attrition |
|------------------|-------|----------------|----------------|
| 0-2500            | 226   | 77             | 34.07          |
| 2500-5000         | 523   | 86             | 16.44          |
| 5000-10000        | 440   | 49             | 11.14          |
| 10000-20000       | 281   | 25             | 8.90           |

```python
plt.figure(figsize=(10, 6))

sns.barplot(income_results, x='MonthlyIncomeGroup', y='% of Attrition')

plt.title('% of Attrition by Monthly Income', fontsize = 14, weight = 'bold')
plt.xlabel('Monthly Income', fontsize = 12)
plt.ylabel('% of Attrition', fontsize = 12)
plt.tight_layout()
```
<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/5cbd7b2c-e491-41da-a87b-28da712622de" />

### Two-Sample t-Test: Attrition vs Monthly Income
α = 0.05

H₀: There is no difference in MonthlyIncome between leavers and stayers.

H₁: There is a difference in MonthlyIncome between leavers and stayers.
```python
left = df[df['Attrition']=="Yes"]['MonthlyIncome']
stayed = df[df['Attrition']=="No"]['MonthlyIncome']

t_stat, p_value = stats.ttest_ind(a=left,
                b=stayed,
                equal_var=False)
```

```python
np.round(p_value, 13)
```
`4e-13`
### Result
On average, employees who left earned $4,787, compared to $6,833 for those who stayed. The chart shows that salary has a strong impact on attrition: 34.1% of employees earning $0–2,500 leave the company, while attrition steadily decreases with higher salaries, reaching only 8.9% in the $10,000–20,000 range. After performing a two-sample t-test, I found that the p-value of 4e-13 strongly rejects H₀, indicating a statistically significant difference in mean MonthlyIncome between leavers and stayers.

## Final Results & Business Recommendations 

### Key Findings
- **Salary is the strongest attrition driver**: 34% of employees earning <2,500 leave, compared to only 9% earning 10,000–20,000.  
- **Early tenure risk**: Most attrition occurs during the first 1–2 years of employment.  
- **Department & role patterns**: Sales department (20.6%) with Sales Representatives (39.8%) shows the highest attrition.  
- **Work-life balance factors**: Frequent business travel and low work-life balance contribute significantly to employee turnover.  

### Business Recommendations
1. **Compensation**
   - Address below-average earners and monitor pay equity across roles to ensure fair compensation.  
2. **Onboarding & Early-Career Support**
   -  Implement targeted retention programs during the first one to two years of employment.
3. **Workload & Travel Management**
   - Introduce flexible policies to reduce attrition among frequent travelers and employees experiencing poor work-life balance.
4. **Ongoing Monitoring**
   - Continuously track attrition trends and evaluate the effectiveness of retention interventions.  

**Outcome:** Implementing these strategies can improve retention, reduce costs associated with turnover, and enhance overall workforce stability.


