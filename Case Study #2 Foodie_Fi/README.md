# 🥑 Case Study #3: Foodie-Fi

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-3/). 

***

## Business Task
Danny and his friends launched a new startup Foodie-Fi and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world.

This case study focuses on using subscription style digital data to answer important business questions on customer journey, payments, and business performances.

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png)

**Table 1: `plans`**

<img width="207" alt="image" src="https://user-images.githubusercontent.com/81607668/135704535-a82fdd2f-036a-443b-b1da-984178166f95.png">

There are 5 customer plans.

- Trial — Customer sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
- Basic plan — Customers have limited access and can only stream their videos and is only available monthly at $9.90.
- Pro plan — Customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

When customers cancel their Foodie-Fi service — they will have a Churn plan record with a null price, but their plan will continue until the end of the billing period.

**Table 2: `subscriptions`**

<img width="245" alt="image" src="https://user-images.githubusercontent.com/81607668/135704564-30250dd9-6381-490a-82cf-d15e6290cf3a.png">

Customer subscriptions show the **exact date** where their specific `plan_id` starts.

If customers downgrade from a pro plan or cancel their subscription — the higher plan will remain in place until the period is over — the `start_date` in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan — the higher plan will take effect straightaway.

When customers churn, they will keep their access until the end of their current billing period, but the start_date will be technically the day they decided to cancel their service.
---
### 1. How many customers has Foodie-Fi ever had?

To determine the count of unique customers for Foodie-Fi, I utilize the `COUNT()` function wrapped around `DISTINCT`.

```sql
SELECT COUNT(DISTINCT customer_id) AS num_of_customers
FROM foodie_fi.subscriptions;
```
| Total_no_of_customers |
| --------------------- |
| 1000  		|

### 2. How many days has each customer visited the restaurant?
``` sql
SELECT YEAR(start_date) * 100 + MONTH(start_date) Month_year,  
		DATENAME(MONTH, start_date) Month,  
		COUNT(DISTINCT customer_id) No_of_trial_plans  
FROM foodie_fi.subscriptions s  
WHERE s.plan_id = 0  
GROUP BY YEAR(start_date) * 100 + MONTH(start_date),  
		DATENAME(MONTH, start_date)  
ORDER BY YEAR(start_date) * 100 + MONTH(start_date);  
```

![2](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/8dfe7a90-757a-4bd1-b438-2a61815625da)

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
``` sql
Select 
	p.plan_name,
	Count(p.plan_id) count
FROM 
	foodie_fi.subscriptions s
	INNER JOIN plans p
		ON p.plan_id = s.plan_id
WHERE
	start_date >= '2021-01-01'
GROUP BY 
	p.plan_name
ORDER BY
	count;
```
![3](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/ff95dbbc-b170-4fc5-a03a-e549da3c4ca9)

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
``` sql
WITH total as(
	SELECT COUNT(DISTINCT s.customer_id) as gross
	FROM foodie_fi.subscriptions s
)

SELECT A.Churn,	
	   ROUND(CAST(A.Churn AS float)/total.gross * 100, 2)
FROM ( 
	Select COUNT(s.customer_id) as Churn
	FROM foodie_fi.subscriptions s
	WHERE plan_id = 4
) A, 
total;
```
![4](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/cab7bc05-ac83-4089-b749-3d2afffcf003)

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
``` sql
WITH imm_churn as (
SELECT COUNT(customer_id) as churned
FROM(
	SELECT sub.customer_id, churn_date, start_date
	FROM (
		SELECT s.customer_id, s.start_date as churn_date
		FROM foodie_fi.subscriptions as s
		WHERE plan_id = 4
	) as a
	inner join foodie_fi.subscriptions as sub
	ON sub.customer_id = a.customer_id
	WHERE plan_id = 0
	) b
WHERE DATEDIFF(DAY, start_date, churn_date) = 7
)
SELECT imm_churn.churned, ROUND(CAST(imm_churn.churned as float)/ A.Churn * 100,2) as total_churned
FROM( 
	Select COUNT(s.customer_id) as Churn
	FROM foodie_fi.subscriptions s
	WHERE plan_id = 4
) A, imm_churn

```
![5](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/1113e696-7846-41b8-82d1-d7af4cccd965)

### 6. What is the number and percentage of customer plans after their initial free trial?
``` sql
   SELECT 
		plan_name, 
		count(customer_id) as no_of_customers
FROM (
	SELECT *
	FROM ( 
		Select s.customer_id, 
			   p.plan_name,
			   ROW_NUMBER () Over(Partition by s.customer_id order by s.start_date) as Row_no
		FROM foodie_fi.subscriptions as s
		INNER JOIN foodie_fi.plans as p
		 ON p.plan_id = s.plan_id
		 ) A
	WHERE Row_no = 2
	) B
GROUP BY plan_name
ORDER BY count(customer_id)
```
![6](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/56c1aeb0-f386-4e31-9ea3-5b232f7cd953)

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
``` sql
WITH CTE AS (
SELECT s.customer_id, plan_name, start_date, ROW_NUMBER() over (partition by customer_id order by start_date DESC)rn
FROM foodie_fi.subscriptions as s
INNER JOIN plans as p
	ON p.plan_id = s.plan_id
WHERE start_date < '2020-12-31'
)
SELECT p.plan_name, 
	COUNT(customer_id) customer_count, 
	ROUND(CAST(COUNT(customer_id) AS FLOAT)/(SELECT count(DISTINCT customer_id) FROM foodie_fi.subscriptions) * 100,1) as percentage
FROM CTE
INNER JOIN plans p
	ON p.plan_name = CTE.plan_name
WHERE rn = 1
GROUP By p.plan_name

-- Option 2
WITH CTE AS (
SELECT s.customer_id, plan_name, start_date, ROW_NUMBER() over (partition by customer_id order by start_date DESC)rn
FROM foodie_fi.subscriptions as s
INNER JOIN plans as p
	ON p.plan_id = s.plan_id
WHERE '2020-12-31' BETWEEN start_date AND DATEADD(MONTH, 1, start_date)
)
```

![7](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/8a7967ed-dd81-4592-bd36-11db843ec27b)

### 8. How many customers have upgraded to an annual plan in 2020?
``` sql
WITH CTE AS (
    SELECT 
        customer_id, 
        plan_name, 
        s.start_date, 
        LAG(p.plan_name) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS lg
    FROM 
        foodie_fi.subscriptions AS s
    INNER JOIN 
        foodie_fi.plans AS p ON p.plan_id = s.plan_id
    WHERE 
        YEAR(s.start_date) = 2020
)
SELECT 
    Count(distinct CTE.customer_id) as No_of_customer
FROM 
    CTE
WHERE 
    lg IN ('basic monthly', 'pro monthly')
    AND plan_name = 'pro annual';
```
![8](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/eecb482e-d63f-4fbe-960c-69b67e605f03)

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
``` sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
), annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
)
-- Find the average of the differences between the start date of a trial plan and a pro annual plan.
SELECT 
  ROUND(
    AVG(DATEDIFF(DAY,trial.trial_date,annual.annual_date))
  ,0) AS avg_days_to_upgrade
FROM trial_plan AS trial
JOIN annual_plan AS annual
  ON trial.customer_id = annual.customer_id;
```
![9](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/97638908-98e6-413d-b9f6-8360548fef90)

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
``` sql
WITH trial_plan AS (
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
), annual_plan AS (
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
), bins AS (
  SELECT 
    CEILING(DATEDIFF(DAY, trial.trial_date, annual.annual_date) / 30.0) AS avg_days_to_upgrade
  FROM trial_plan AS trial
  JOIN annual_plan AS annual
    ON trial.customer_id = annual.customer_id
)
  
SELECT 
  CONCAT(((avg_days_to_upgrade - 1) * 30), ' - ', (avg_days_to_upgrade * 30), ' days') AS bucket, 
  COUNT(*) AS num_of_customers
FROM bins
GROUP BY avg_days_to_upgrade
ORDER BY avg_days_to_upgrade;
```
![10](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/6d960f4f-45c2-4ef5-b084-e85768213f31)

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
``` sql
With CTE as (
	SELECT customer_id, plan_name as current_plan, LAG(plan_name) over(partition by customer_id order by start_date) as early_plan
	FROM 
	foodie_fi.subscriptions as s
	INNER JOIN 
			foodie_fi.plans AS p ON p.plan_id = s.plan_id
	WHERE YEAR(start_date) = 2020
)

Select COUNT(distinct customer_id)
from CTE
where early_plan = 'pro monthly' and current_plan = 'basic monthly'
```
![11](https://github.com/Prafulla-KumarB/8-Week-SQL-Challenge/assets/92779688/51cc6415-765c-40d8-99d2-fbbcaf343b81)


