# Data Transformations Guide

Complete step-by-step guide for all Power Query transformations applied to the HR dataset.

## Table of Contents
1. [Initial Data Load](#initial-data-load)
2. [Data Type Conversions](#data-type-conversions)
3. [Text Cleaning](#text-cleaning)
4. [Handling Null Values](#handling-null-values)
5. [Calculated Columns](#calculated-columns)
6. [Column Removal](#column-removal)
7. [Final Quality Checks](#final-quality-checks)

---

## Initial Data Load

### Step 1: Import CSV File

**UI Steps:**
1. Home → Get Data → Text/CSV
2. Navigate to `HRDataset_v14.csv`
3. Click "Transform Data" to open Power Query Editor

**M Code:**
```m
let
    Source = Csv.Document(File.Contents("C:\Path\To\HRDataset_v14.csv"),[Delimiter=",", Columns=36, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true])
in
    #"Promoted Headers"
```

### Step 2: Verify Column Names

Check that all 36 columns are correctly named:
- Employee_Name, EmpID, MarriedID, MaritalStatusID, GenderID, EmpStatusID, DeptID, PerfScoreID, FromDiversityJobFairID, Salary, Termd, PositionID, Position, State, Zip, DOB, Sex, MaritalDesc, CitizenDesc, HispanicLatino, RaceDesc, DateofHire, DateofTermination, TermReason, EmploymentStatus, Department, ManagerName, ManagerID, RecruitmentSource, PerformanceScore, EngagementSurvey, EmpSatisfaction, SpecialProjectsCount, LastPerformanceReview_Date, DaysLateLast30, Absences

---

## Data Type Conversions

### Step 3: Convert Date Columns

**DOB (Date of Birth):**

**UI Steps:**
1. Select DOB column
2. Transform → Data Type → Date

**M Code:**
```m
#"Changed Type DOB" = Table.TransformColumnTypes(#"Promoted Headers",{{"DOB", type date}})
```

**DateofHire:**

**UI Steps:**
1. Select DateofHire column
2. Transform → Data Type → Date

**M Code:**
```m
#"Changed Type Hire" = Table.TransformColumnTypes(#"Changed Type DOB",{{"DateofHire", type date}})
```

**DateofTermination:**

**UI Steps:**
1. Select DateofTermination column
2. Transform → Data Type → Date
3. Note: Nulls are preserved (active employees)

**M Code:**
```m
#"Changed Type Term" = Table.TransformColumnTypes(#"Changed Type Hire",{{"DateofTermination", type date}})
```

**LastPerformanceReview_Date:**

**UI Steps:**
1. Select LastPerformanceReview_Date column
2. Transform → Data Type → Date

**M Code:**
```m
#"Changed Type Review" = Table.TransformColumnTypes(#"Changed Type Term",{{"LastPerformanceReview_Date", type date}})
```

### Step 4: Convert Numeric Columns

**ManagerID (Float to Integer):**

**UI Steps:**
1. Select ManagerID column
2. Transform → Data Type → Whole Number

**M Code:**
```m
#"Changed Type ManagerID" = Table.TransformColumnTypes(#"Changed Type Review",{{"ManagerID", Int64.Type}})
```

---

## Text Cleaning

### Step 5: Trim All Text Columns

**Select Multiple Columns:**

**UI Steps:**
1. Hold Ctrl and select:
   - Employee_Name
   - Sex
   - MaritalDesc
   - Position
   - Department
   - ManagerName
   - CitizenDesc
   - RaceDesc
   - TermReason
   - EmploymentStatus
   - RecruitmentSource
   - PerformanceScore
2. Transform → Format → Trim

**M Code:**
```m
#"Trimmed Text" = Table.TransformColumns(#"Changed Type ManagerID",
    {
        {"Employee_Name", Text.Trim, type text},
        {"Sex", Text.Trim, type text},
        {"MaritalDesc", Text.Trim, type text},
        {"Position", Text.Trim, type text},
        {"Department", Text.Trim, type text},
        {"ManagerName", Text.Trim, type text},
        {"CitizenDesc", Text.Trim, type text},
        {"RaceDesc", Text.Trim, type text},
        {"TermReason", Text.Trim, type text},
        {"EmploymentStatus", Text.Trim, type text},
        {"RecruitmentSource", Text.Trim, type text},
        {"PerformanceScore", Text.Trim, type text}
    }
)
```

### Step 6: Clean Extra Spaces

For columns with excessive middle spacing (like Department):

**UI Steps:**
1. Select Department column
2. Transform → Format → Clean

**M Code:**
```m
#"Cleaned Text" = Table.TransformColumns(#"Trimmed Text",
    {
        {"Department", Text.Clean, type text},
        {"Employee_Name", Text.Clean, type text}
    }
)
```

---

## Handling Null Values

### Step 7: Handle DateofTermination Nulls

**Keep nulls as-is** (they represent active employees)

No transformation needed - nulls are meaningful data.

### Step 8: Handle ManagerID Nulls

**Option A: Keep Nulls**
```m
// No transformation - leave as is
```

**Option B: Replace with "No Manager"**
```m
#"Replaced ManagerID Nulls" = Table.ReplaceValue(
    #"Cleaned Text",
    null,
    0,
    Replacer.ReplaceValue,
    {"ManagerID"}
)
```

---

## Calculated Columns

### Step 9: Add Age Column

**UI Steps:**
1. Add Column → Custom Column
2. Name: `Age`
3. Formula:

**M Code:**
```m
#"Added Age" = Table.AddColumn(
    #"Cleaned Text", 
    "Age", 
    each Number.RoundDown(Duration.Days(DateTime.LocalNow() - [DOB]) / 365.25),
    Int64.Type
)
```

### Step 10: Add Tenure Column

**UI Steps:**
1. Add Column → Custom Column
2. Name: `Tenure (Years)`
3. Formula:

**M Code:**
```m
#"Added Tenure" = Table.AddColumn(
    #"Added Age", 
    "Tenure (Years)", 
    each Number.RoundDown(Duration.Days(DateTime.LocalNow() - [DateofHire]) / 365.25),
    Int64.Type
)
```

### Step 11: Add Is Active Column

**UI Steps:**
1. Add Column → Conditional Column
2. Name: `Is Active`
3. If `Termd` = 0, then "Active", else "Terminated"

**M Code:**
```m
#"Added Is Active" = Table.AddColumn(
    #"Added Tenure", 
    "Is Active", 
    each if [Termd] = 0 then "Active" else "Terminated",
    type text
)
```

**Alternative (Boolean):**
```m
#"Added Is Active Boolean" = Table.AddColumn(
    #"Added Tenure", 
    "Is Active", 
    each [Termd] = 0,
    type logical
)
```

### Step 12: Add Age Group Column

**UI Steps:**
1. Add Column → Custom Column
2. Name: `Age Group`
3. Formula:

**M Code:**
```m
#"Added Age Group" = Table.AddColumn(
    #"Added Is Active", 
    "Age Group", 
    each if [Age] < 30 then "20-29"
         else if [Age] < 40 then "30-39"
         else if [Age] < 50 then "40-49"
         else if [Age] < 60 then "50-59"
         else "60+",
    type text
)
```

### Step 13: Add Salary Band Column

**UI Steps:**
1. Add Column → Custom Column
2. Name: `Salary Band`
3. Formula:

**M Code:**
```m
#"Added Salary Band" = Table.AddColumn(
    #"Added Age Group", 
    "Salary Band", 
    each if [Salary] < 50000 then "< $50K"
         else if [Salary] < 75000 then "$50K - $75K"
         else if [Salary] < 100000 then "$75K - $100K"
         else "$100K+",
    type text
)
```

---

## Column Removal

### Step 14: Remove Redundant ID Columns

**Columns to Remove:**
- MarriedID (have MaritalDesc)
- MaritalStatusID (have MaritalDesc)
- GenderID (have Sex)
- EmpStatusID (have EmploymentStatus)
- DeptID (have Department)
- PerfScoreID (have PerformanceScore)

**UI Steps:**
1. Select columns (hold Ctrl)
2. Right-click → Remove Columns

**M Code:**
```m
#"Removed Columns" = Table.RemoveColumns(
    #"Added Salary Band",
    {"MarriedID", "MaritalStatusID", "GenderID", "EmpStatusID", "DeptID", "PerfScoreID"}
)
```

### Step 15: Reorder Columns (Optional)

**UI Steps:**
1. Drag columns to preferred order
2. Suggested order: Employee info → Dates → Performance → Compensation

**M Code:**
```m
#"Reordered Columns" = Table.ReorderColumns(
    #"Removed Columns",
    {"Employee_Name", "EmpID", "Position", "PositionID", "Department", "ManagerName", "ManagerID", 
     "Sex", "MaritalDesc", "DOB", "Age", "Age Group", "DateofHire", "Tenure (Years)", 
     "EmploymentStatus", "Is Active", "Termd", "DateofTermination", "TermReason",
     "Salary", "Salary Band", "PerformanceScore", "EngagementSurvey", "EmpSatisfaction",
     "RecruitmentSource", "FromDiversityJobFairID", "CitizenDesc", "HispanicLatino", "RaceDesc",
     "SpecialProjectsCount", "LastPerformanceReview_Date", "DaysLateLast30", "Absences", "State", "Zip"}
)
```

---

## Final Quality Checks

### Step 16: Replace N/A Values in TermReason

**UI Steps:**
1. Select TermReason column
2. Transform → Replace Values
3. Value to Find: `N/A-StillEmployed`
4. Replace With: `null`

**M Code:**
```m
#"Replaced N/A" = Table.ReplaceValue(
    #"Reordered Columns",
    "N/A-StillEmployed",
    null,
    Replacer.ReplaceValue,
    {"TermReason"}
)
```

### Step 17: Validate Data Types

**Check all columns have correct types:**
- Text: Employee_Name, Position, Department, etc.
- Date: DOB, DateofHire, DateofTermination, LastPerformanceReview_Date
- Whole Number: EmpID, PositionID, Age, Tenure, Salary, Absences, etc.
- Decimal: EngagementSurvey
- Boolean/Text: Is Active

### Step 18: Check for Errors

**UI Steps:**
1. View → Column Quality
2. View → Column Distribution
3. View → Column Profile
4. Check for errors, nulls, and data quality issues

---

## Complete M Code

Here's the complete Power Query transformation code:

```m
let
    Source = Csv.Document(File.Contents("HRDataset_v14.csv"),[Delimiter=",", Columns=36, Encoding=65001]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    
    // Convert date columns
    #"Changed Type Dates" = Table.TransformColumnTypes(#"Promoted Headers",{
        {"DOB", type date}, 
        {"DateofHire", type date}, 
        {"DateofTermination", type date}, 
        {"LastPerformanceReview_Date", type date}
    }),
    
    // Convert numeric columns
    #"Changed Type Numbers" = Table.TransformColumnTypes(#"Changed Type Dates",{
        {"ManagerID", Int64.Type}
    }),
    
    // Trim text columns
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type Numbers", {
        {"Employee_Name", Text.Trim}, {"Sex", Text.Trim}, {"MaritalDesc", Text.Trim},
        {"Position", Text.Trim}, {"Department", Text.Trim}, {"ManagerName", Text.Trim},
        {"CitizenDesc", Text.Trim}, {"RaceDesc", Text.Trim}, {"TermReason", Text.Trim},
        {"EmploymentStatus", Text.Trim}, {"RecruitmentSource", Text.Trim}, {"PerformanceScore", Text.Trim}
    }),
    
    // Clean extra spaces
    #"Cleaned Text" = Table.TransformColumns(#"Trimmed Text", {
        {"Department", Text.Clean}, {"Employee_Name", Text.Clean}
    }),
    
    // Add calculated columns
    #"Added Age" = Table.AddColumn(#"Cleaned Text", "Age", 
        each Number.RoundDown(Duration.Days(DateTime.LocalNow() - [DOB]) / 365.25), Int64.Type),
    
    #"Added Tenure" = Table.AddColumn(#"Added Age", "Tenure (Years)", 
        each Number.RoundDown(Duration.Days(DateTime.LocalNow() - [DateofHire]) / 365.25), Int64.Type),
    
    #"Added Is Active" = Table.AddColumn(#"Added Tenure", "Is Active", 
        each if [Termd] = 0 then "Active" else "Terminated", type text),
    
    #"Added Age Group" = Table.AddColumn(#"Added Is Active", "Age Group", 
        each if [Age] < 30 then "20-29"
             else if [Age] < 40 then "30-39"
             else if [Age] < 50 then "40-49"
             else if [Age] < 60 then "50-59"
             else "60+", type text),
    
    #"Added Salary Band" = Table.AddColumn(#"Added Age Group", "Salary Band", 
        each if [Salary] < 50000 then "< $50K"
             else if [Salary] < 75000 then "$50K - $75K"
             else if [Salary] < 100000 then "$75K - $100K"
             else "$100K+", type text),
    
    // Remove redundant columns
    #"Removed Columns" = Table.RemoveColumns(#"Added Salary Band",
        {"MarriedID", "MaritalStatusID", "GenderID", "EmpStatusID", "DeptID", "PerfScoreID"}),
    
    // Replace N/A values
    #"Replaced N/A" = Table.ReplaceValue(#"Removed Columns", "N/A-StillEmployed", null, 
        Replacer.ReplaceValue, {"TermReason"})
in
    #"Replaced N/A"
```

---

## Tips & Best Practices

1. **Always transform data in Power Query** (not in DAX when possible)
2. **Keep original columns** until transformation is verified
3. **Use Column Profile** to check data quality
4. **Document your steps** with comments in Advanced Editor
5. **Test with sample data** before applying to full dataset
6. **Use Table.TransformColumns** for bulk operations
7. **Handle nulls appropriately** - they may be meaningful
8. **Add calculated columns** that will be used for filtering/slicing

---

## Common Issues & Solutions

**Issue**: Date columns showing as text
- **Solution**: Ensure date format in CSV matches Power BI locale settings

**Issue**: Extra spaces not removed
- **Solution**: Use Clean (not just Trim) for middle spaces

**Issue**: Age calculation showing decimal
- **Solution**: Use Number.RoundDown and set type to Int64.Type

**Issue**: Errors in calculated columns
- **Solution**: Check for nulls, handle with if-then-else logic

---

**Last Updated**: March 2026