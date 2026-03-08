# DAX Measures Documentation

Complete reference for all DAX measures used in the HR Analytics Dashboard.

## Table of Contents
- [Workforce Metrics](#workforce-metrics)
- [Turnover & Retention](#turnover--retention)
- [Compensation](#compensation)
- [Performance](#performance)
- [Recruitment](#recruitment)
- [Time-Based Calculations](#time-based-calculations)
- [Advanced Analytics](#advanced-analytics)

---

## Workforce Metrics

### Total Employees
```dax
Total Employees = COUNTROWS('HR Data')
```
**Description**: Counts all employee records in the dataset.

### Active Employees
```dax
Active Employees = 
CALCULATE(
    COUNTROWS('HR Data'),
    'HR Data'[Termd] = 0
)
```
**Description**: Counts only active employees (where Termd = 0).

### Terminated Employees
```dax
Terminated Employees = 
CALCULATE(
    COUNTROWS('HR Data'),
    'HR Data'[Termd] = 1
)
```
**Description**: Counts terminated employees (where Termd = 1).

### Headcount by Department
```dax
Dept Headcount = 
CALCULATE(
    COUNTROWS('HR Data'),
    ALLEXCEPT('HR Data', 'HR Data'[Department])
)
```
**Description**: Automatically calculates headcount per department when used in visuals.

---

## Turnover & Retention

### Turnover Rate %
```dax
Turnover Rate % = 
DIVIDE(
    [Terminated Employees],
    [Total Employees],
    0
) * 100
```
**Description**: Percentage of employees who left the organization.

### Retention Rate %
```dax
Retention Rate % = 
DIVIDE(
    [Active Employees],
    [Total Employees],
    0
) * 100
```
**Description**: Percentage of employees still with the organization.

### Voluntary Turnover %
```dax
Voluntary Turnover % = 
VAR Voluntary = 
    CALCULATE(
        COUNTROWS('HR Data'),
        'HR Data'[EmploymentStatus] = "Voluntarily Terminated"
    )
RETURN
DIVIDE(Voluntary, [Total Employees], 0) * 100
```
**Description**: Percentage who left voluntarily (resignations).

### Involuntary Turnover %
```dax
Involuntary Turnover % = 
VAR Involuntary = 
    CALCULATE(
        COUNTROWS('HR Data'),
        'HR Data'[EmploymentStatus] = "Terminated for Cause"
    )
RETURN
DIVIDE(Involuntary, [Total Employees], 0) * 100
```
**Description**: Percentage terminated for cause.

### Annual Turnover Rate
```dax
Annual Turnover Rate % = 
VAR TerminatedLastYear = 
    CALCULATE(
        COUNTROWS('HR Data'),
        'HR Data'[DateofTermination] >= DATE(YEAR(TODAY())-1, MONTH(TODAY()), DAY(TODAY())),
        'HR Data'[DateofTermination] <= TODAY()
    )
VAR AvgEmployees = [Total Employees]
RETURN
DIVIDE(TerminatedLastYear, AvgEmployees, 0) * 100
```
**Description**: Turnover rate for the last 12 months.

---

## Compensation

### Average Salary
```dax
Avg Salary = AVERAGE('HR Data'[Salary])
```
**Description**: Mean salary across all employees.

### Median Salary
```dax
Median Salary = MEDIAN('HR Data'[Salary])
```
**Description**: Median salary value (middle point).

### Min Salary
```dax
Min Salary = MIN('HR Data'[Salary])
```
**Description**: Lowest salary in the dataset.

### Max Salary
```dax
Max Salary = MAX('HR Data'[Salary])
```
**Description**: Highest salary in the dataset.

### Total Payroll
```dax
Total Payroll = SUM('HR Data'[Salary])
```
**Description**: Sum of all salaries (annual payroll cost).

### Avg Salary by Performance
```dax
Avg Salary by Performance = 
CALCULATE(
    [Avg Salary],
    ALLEXCEPT('HR Data', 'HR Data'[PerformanceScore])
)
```
**Description**: Average salary segmented by performance rating.

### Salary Range
```dax
Salary Range = [Max Salary] - [Min Salary]
```
**Description**: Difference between highest and lowest salary.

### Salary Above Average
```dax
Count Above Avg Salary = 
CALCULATE(
    COUNTROWS('HR Data'),
    'HR Data'[Salary] > [Avg Salary]
)
```
**Description**: Number of employees earning above average.

---

## Performance

### Average Engagement Score
```dax
Avg Engagement = AVERAGE('HR Data'[EngagementSurvey])
```
**Description**: Mean engagement survey score.

### Average Satisfaction
```dax
Avg Satisfaction = AVERAGE('HR Data'[EmpSatisfaction])
```
**Description**: Mean employee satisfaction rating.

### High Performers Count
```dax
High Performers = 
CALCULATE(
    COUNTROWS('HR Data'),
    'HR Data'[PerformanceScore] = "Exceeds"
)
```
**Description**: Count of employees with "Exceeds" performance rating.

### Low Performers Count
```dax
Low Performers = 
CALCULATE(
    COUNTROWS('HR Data'),
    OR(
        'HR Data'[PerformanceScore] = "Needs Improvement",
        'HR Data'[PerformanceScore] = "PIP"
    )
)
```
**Description**: Count of employees needing improvement or on PIP.

### High Performer %
```dax
High Performer % = 
DIVIDE(
    [High Performers],
    [Total Employees],
    0
) * 100
```
**Description**: Percentage of workforce exceeding expectations.

### Engagement Target Achievement
```dax
Engagement vs Target = 
VAR Target = 4.5
VAR Actual = [Avg Engagement]
RETURN
IF(Actual >= Target, "Met", "Below Target")
```
**Description**: Compares actual engagement to target (4.5).

---

## Recruitment

### Hires This Year
```dax
Hires This Year = 
CALCULATE(
    COUNTROWS('HR Data'),
    YEAR('HR Data'[DateofHire]) = YEAR(TODAY())
)
```
**Description**: Number of employees hired in current year.

### Diversity Hires
```dax
Diversity Hires = 
CALCULATE(
    COUNTROWS('HR Data'),
    'HR Data'[FromDiversityJobFairID] = 1
)
```
**Description**: Count of hires from diversity job fair.

### Diversity Hire %
```dax
Diversity Hire % = 
DIVIDE(
    [Diversity Hires],
    [Total Employees],
    0
) * 100
```
**Description**: Percentage of workforce from diversity recruitment.

### Top Recruitment Source
```dax
Top Source = 
CALCULATE(
    VALUES('HR Data'[RecruitmentSource]),
    TOPN(1, ALL('HR Data'[RecruitmentSource]), COUNTROWS('HR Data'), DESC)
)
```
**Description**: Most effective recruitment channel.

---

## Time-Based Calculations

### Average Tenure (Years)
```dax
Avg Tenure Years = 
AVERAGEX(
    'HR Data',
    DATEDIFF('HR Data'[DateofHire], TODAY(), YEAR)
)
```
**Description**: Average length of service in years.

### Average Age
```dax
Avg Age = 
AVERAGEX(
    'HR Data',
    DATEDIFF('HR Data'[DOB], TODAY(), YEAR)
)
```
**Description**: Mean age of workforce.

### Days Since Last Review
```dax
Days Since Last Review = 
AVERAGEX(
    'HR Data',
    DATEDIFF('HR Data'[LastPerformanceReview_Date], TODAY(), DAY)
)
```
**Description**: Average days since last performance review.

---

## Advanced Analytics

### Attrition Risk Score
```dax
Attrition Risk = 
VAR LowEngagement = IF([EngagementSurvey] < 3, 1, 0)
VAR LowSatisfaction = IF([EmpSatisfaction] < 3, 1, 0)
VAR HighAbsences = IF([Absences] > 10, 1, 0)
VAR LowPerformance = IF([PerformanceScore] IN {"Needs Improvement", "PIP"}, 1, 0)
RETURN
LowEngagement + LowSatisfaction + HighAbsences + LowPerformance
```
**Description**: Risk score (0-4) based on multiple factors.

### High Risk Employees
```dax
High Risk Count = 
CALCULATE(
    COUNTROWS('HR Data'),
    [Attrition Risk] >= 3
)
```
**Description**: Employees with risk score 3 or higher.

### Gender Pay Gap
```dax
Gender Pay Gap % = 
VAR MaleSalary = 
    CALCULATE([Avg Salary], 'HR Data'[Sex] = "M")
VAR FemaleSalary = 
    CALCULATE([Avg Salary], 'HR Data'[Sex] = "F")
RETURN
DIVIDE(MaleSalary - FemaleSalary, MaleSalary, 0) * 100
```
**Description**: Percentage difference in average male vs female salary.

### Span of Control
```dax
Avg Span of Control = 
AVERAGEX(
    VALUES('HR Data'[ManagerName]),
    CALCULATE(COUNTROWS('HR Data'))
)
```
**Description**: Average number of direct reports per manager.

### Absence Rate
```dax
Absence Rate = 
VAR TotalAbsences = SUM('HR Data'[Absences])
VAR TotalEmployees = [Total Employees]
RETURN
DIVIDE(TotalAbsences, TotalEmployees, 0)
```
**Description**: Average absences per employee.

---

## Formatting Tips

### Currency Format
For salary measures, set format to: **$ #,##0**

### Percentage Format
For rate measures, set format to: **0.0%** or create calculated measure:
```dax
Turnover Rate Formatted = 
FORMAT([Turnover Rate %], "0.0") & "%"
```

### Conditional Formatting
Use measures as rules for conditional formatting:
```dax
Turnover Color = 
SWITCH(
    TRUE(),
    [Turnover Rate %] < 15, "Green",
    [Turnover Rate %] < 25, "Yellow",
    "Red"
)
```

---

## Best Practices

1. **Use DIVIDE** instead of `/` to handle division by zero
2. **Use CALCULATE** to modify filter context
3. **Use VAR** for complex calculations to improve readability
4. **Name measures clearly** - include units (%, $, Count)
5. **Add descriptions** to measures in the model view
6. **Group related measures** in display folders
7. **Format measures appropriately** (currency, percentage, decimal places)

---

## Testing Measures

Always verify your measures:
```dax
// Test measure
Test Total = 
VAR Active = [Active Employees]
VAR Terminated = [Terminated Employees]
VAR Total = [Total Employees]
RETURN
IF(Active + Terminated = Total, "PASS", "FAIL")
```

---

**Last Updated**: March 2026