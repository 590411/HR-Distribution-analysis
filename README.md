# Human Resource  Employee Distribution  Analysis

# Human resource analysis

## Company activity/ field : Retail Company - Head Rest Ltd

## Table of Content
-  [Project Overview](#project-overview)
-  [Data Sources](#data-sources)
-  [Tools](#tools)
-  [Data cleaning ](#data-cleaning )
-  [Exploratory data analysis](#exploratory-data-analysis)
-  [Data analysis](#data-analysis)
-  [Findings](#findings)
  
### Project overview
---

---- **Delivrables**

   - Create a data model for analysis
   - A dashboard in which we see the distribution per gender , race , age , age group distribution, gender distibution per department , age distibution per gender , termination disbtribution per department
   - A chart regarding the evolution of employees nunmber




  

 ### Data sources
 ---

*Company  Data  on  Human resource operations,the dataset use for this analysis is "Human Resources.csv"  file . we have data on birthdate , department ,hire_date , location_state ........*

### Tools
---

  - Excel-power query : ETL
  - SQL : Data cleaning
  - MYSQL : Transformation and loading
  - Power BI : Data model and visuaization
 

### Data cleaning 
---

- From the excel we see that , the data type are text and date
- The birthday , hire_date and termdate  variables are dates but have different format , so we have to  make sure that there all have the same format
- We have termdate which are in the future, which is not normal

-- Creating the database
```
create database if not exists human_resource ;
```

-- selecting the database to be use
```
use human_resource ;
```

-- modifying the column name from  ï»¿id to emp_ID
```
alter table hr
change column ï»¿id emp_ID varchar(20) null;
```

-- Checking the data information (fields . datatype, key, null, default , extra) 
```
describe hr ;
```

-- Remove the update restrictions
```
set sql_safe_updates = 0;
```

--As soon as you  have finish with all your updates , you have to set it back to 1 , to secure back your database (is a security feature that is protecting your database)
```
set sql_safe_updates = 1;
```

-- Harmonizing the birthdate column  by Formatting 
```
update hr
set birthdate = case
when birthdate like '%/%' then  date_format(str_to_date( birthdate, '%m/%d/%Y'), '%Y-%m-%d')
when birthdate like '%-%' then  date_format(str_to_date( birthdate, '%m-%d-%Y'), '%Y-%m-%d')
Else Null
end ;
```

-- Modify the datatype of  birthdate column from text to date
```
alter table hr
modify column birthdate date ;
```

-- Harmonizing the hire_date column  by Formatting 
```
update hr 
set hire_date = case
when hire_date like '%/%' then  date_format(str_to_date( hire_date, '%m/%d/%Y'), '%Y-%m-%d')
when hire_date like '%-%' then  date_format(str_to_date( hire_date, '%m-%d-%Y'), '%Y-%m-%d')
Else Null
end ;
```

-- Modify the datatype of hire_date column from text to date
```
alter table hr
modify column hire_date date ;
```

-- Harmonizing and formatting the values of the termdate column from datetime to date
```
update hr
set termdate = date(str_to_date( termdate, '%Y-%m-%d %H:%i:%s UTC'))
where termdate is not null and termdate != '' ;
```

-- Modify the datatype of termdate column from text to date
```
alter table hr
modify column termdate date ;
```

-- Adding a column (age) in the table for analysis purpose
```
alter table  hr
add column age int ;
```

-- calculating the age of employees
```
update hr
set age = timestampdiff(YEAR,birthdate,curdate()) ;
```

-- Identifying the youngest and the oldest  employee 
```
SELECT 
    MIN(age) AS youngest, MAX(age) AS oldest
FROM
    hr;
```

-- Identifying the employees that has less than 23 years 
```
select 
 count(*) 
from hr
 where age < 23 ;
```

-- Gender breakdown of employees in the company above 23 years and still working in the company
```
select 
 gender , 
 count(*) as count
from hr
where age >= 23 and termdate ='' 
 group by gender ;
```

-- Race/ethnicity breakdown of employees in the company above 23 years and still working in the company
```
 select 
 race, 
 count(*) as count
from hr
where age >= 23 and termdate ='' 
group by race 
order by count(*) desc ; 
```

-- Age distribution of employees in the company
 ```
 SELECT 
    MIN(age) AS youngest, MAX(age) AS oldest
FROM
    hr
WHERE
    age >= 23 AND termdate = '';
 ```   
------
```
select 
case
 when age>=23 and age<= 24 then '23-24'
 when age>=25 and age<= 34 then '25-34'
 when age>=35 and age<= 44 then '35-44'
 when age>=35 and age<= 44 then '35-44'
 when age>=45 and age<= 54 then '45-54'
 when age>=55 and age<= 65 then '55-65'
 else '65+'
 end as age_group ,
 count(*) as count
 from hr
 where age >= 23 AND termdate = ''
 group by age_group
 order by age_group ;
```

-- Age-gender distribution of employees in the company above 23 years and still working in the company

```
select 
case
 when age>=23 and age<= 24 then '23-24'
 when age>=25 and age<= 34 then '25-34'
 when age>=35 and age<= 44 then '35-44'
 when age>=35 and age<= 44 then '35-44'
 when age>=45 and age<= 54 then '45-54'
 when age>=55 and age<= 65 then '55-65'
 else '65+'
 end as age_group ,
 gender,
 count(*) as count
 from hr
 where age >= 23 AND termdate = ''
 group by age_group , gender
 order by age_group , gender ;
```

-- How many employees work at headquaters versus remote location, above 23 years and still working in the company
 ```
 select 
 location,
 count(*) as count
 from hr
 where age >= 23 AND termdate = ''
 group by location ;
```

-- Average length of employement for employees who have been termited above 23 years
 ```
select 
round(avg(datediff(termdate, hire_date))/365,0) as avg_length_employement
from hr 
where termdate <= curdate() and termdate <> '' and  age >= 23 ;
```

-- How does the gender distribution varies across departments (23 years and still working in the company)
```
 select 
 department , gender , count(*)  as count
 from hr 
where age >= 23 AND termdate = ''
 group by gender , department 
 order by department ;
```

 -- Distribution of job titles across the company (23 years and still working in the company)
```
  select 
 jobtitle ,  count(*)  as count
 from hr 
where age >= 23 AND termdate = ''
 group by  jobtitle
 order by jobtitle  desc;
```

-- Which department has the highest turnover rate
 ```
 SELECT 
    department,
    total_count,
    terminated_count,
    terminated_count / total_count AS termination_rate
FROM
    (SELECT 
        department,
            COUNT(*) AS total_count,
            SUM(CASE
                WHEN termdate <> '' AND termdate <= CURDATE() THEN 1
                ELSE 0
            END) AS terminated_count
    FROM
        hr
    WHERE
        age >= 23
    GROUP BY department) AS subquery
    order by termination_rate desc ;

```

 -- Distribution of employees across locations by state (23 years and still working in the company)
```
SELECT 
    location_state, COUNT(*) AS count
FROM
    hr
WHERE
    age >= 23 AND termdate = ''
GROUP BY location_state
ORDER BY count DESC;
```

-- How has the company's employee count changed over time based on hire and term dates
```
SELECT 
    year,
    hires,
    terminations,
    hires - terminations AS net_change,
    ROUND((hires - terminations) / hires * 100, 2) AS net_change_percent
FROM
    (SELECT 
        YEAR(hire_date) as year, 
            COUNT(*) AS hires,
            SUM(CASE
                WHEN termdate != '' AND termdate <= CURDATE() THEN 1
                ELSE 0
            END) AS terminations
    FROM
        hr
    WHERE
        age >= 23
    GROUP BY YEAR(hire_date)) AS subquery
ORDER BY YEAR ASC;
```

-- Tenure distribution for each department
```
SELECT 
    department,
    ROUND(AVG(DATEDIFF(termdate, hire_date) / 365),
            0) AS avg_tenure
FROM
    hr
WHERE
    termdate <= CURDATE() AND termdate <> ''
        AND age >= 23
GROUP BY department;

```



### Exploratory data analysis
---

EDA involved exploring  the sales data to  answer key question , such as :

  - What is the overrall  sales trends ?
  - what is the Sales units, growth sales and margin per store  ?
  - what is the Sales units, growth sales and margin per category  ?
  - what is the Sales units, growth sales and margin per brand  ?

### Data analysis
---

```DAX
To create the model , the source data has been normalized  in order to Reduced storage space , Easier maintenance and Improved query speed . The normalization gave
	  - One fact table ; Sales Fact Table
	  - Five dimension tables  ; Product Dimension Table , Store Dimension Table , Date Dimension Table , Manager Dimension Table and Commission Dimension
        Table
	  - Created relationship between Sales Fact Table and dimension tables to build the model
	  - Creating implicit measures

--Measure to calculate units sold :: Units:=sum(Sales[Units Sold])
--Measure to calculate Margin Amount :: MarginAmt:=sum([MarginDollars])
--Measure to calculate sales :: Sales:=SUMX(Sales,[Units Sold]*[UnitPrice])
--Measure to calculate margin percentage :: MarginPct:=[MarginAmt]/[Sales]
--Measure to calculate sales days  ::  SalesDays:=DISTINCTCOUNT([DateID])
--Measure to calculate sales per days :: SalesPerDay:=[Sales]/[SalesDays]
--Calculating Sales percentage of total of product :: Pdt_Sales_PCT:=[Sales]/CALCULATE([Sales],ALL(Dim_Products))
--Calculating Sales percentage of total of managers  :: Mger_sales_PCT:= [Sales] / CALCULATE([Sales],ALL(Dim_Managers))
--Calculating Sales percentage of total of days  :: Day_Sales_PCT:=
   var TotalSales = CALCULATE([Sales],ALLSELECTED(Dim_Dates))
   var Daysales =[Sales]
   return  TotalSales /  Daysales
--calculating promotion sales  :: PromoSales:=CALCULATE([Sales],Sales[Promo]<>"")
--Calculating promotion sales percentage :: Promo_sales_PCT:=[PromoSales] /[Sales]
--Calculating sales of previous year :: SalesPY:=CALCULATE([Sales],SAMEPERIODLASTYEAR(Dim_Dates[Date]))
--Calculating sales growth :: SalesGrowth:=IFERROR([Sales]/[SalesPY]-1, "NA")
--Calculating margin growth ::MarginGrowth:=
    var MarginPY = CALCULATE([MarginPct],SAMEPERIODLASTYEAR(Dim_Dates[Date])) 
    return [MarginPct] - MarginPY		
--Calculating sales per day YTD :: SalesPerdayYTD:=CALCULATE([SalesPerDay],DATESYTD(Dim_Dates[Date]))



 ##Function use
	 
		-- Creating a calculated column for the store size using the switch function :: =SWITCH(Dim_Stores[Store Type],"SM","Small","MED","Medium","WAREHOUSE","WAREHOUSE","OTHER")
			
	  - Other calculation
			
		-- Creating a calculated column for the MarginDollar  :: =[Units Sold]*[UnitPrice]*[RawMargin]
		-- Creating a calculated commission column in the sales table :: =RELATED(Dim_Commission[Commission])*[Units Sold]*[UnitPrice]	
		
## Renaming of  Measures name in pivot table
		
		   YoY sales(Salegrowth) , means year over year sales
		   pdt_sales_PCT share(share) in the business overview
		   Margin(MarginPct)
		   MarginGrowth(Yoy Margin)
		   promo_sales_PCT (Promo%)
		   Day_Sales_PCT(share)in the store performance
		   SalesPerDay(Sales/Day)
		   SalesPerdayYTD (Sales/day YTD)
		   SalesDays(days worked)

		
```

### Findings
---

-- On the business over view we are able to dive in the performance by store , category and brand of bed that the company sells.
 
-- On the store  performance overview , we are able to see which day of the week generated the most sale , performance of each category on the business . 
   Also we can assess the performance of each manager responsible for the store at any given period
 

💻💻  

