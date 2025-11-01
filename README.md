# Cheat_sql_query

1. Alias can be only used in order by clause only.
2. 'Having' is used only with the aggregrate function not 'where' clause. e.g having avg(salary) > 3000.

## Offset

Q. How do you fetch the 6th highest salary from a table?

A. SELECT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 5;  
  
    -- OFFSET 5 skips top 5 salaries   -- LIMIT 1 returns the 6th salary

## Thumb Rule for Group by:
Every column in the SELECT list must be either:
  1. Included in the GROUP BY, or
  2. Used inside an aggregate function (SUM, AVG, COUNT, MAX, MIN).


## Group By Function

1. Type-1: SELECT department, SUM(salary) FROM employees GROUP BY department;
2. Type-2: SELECT SUM(salary) FROM employees GROUP BY department;

   Type-1 and Type-2 works as the department is there is select statement then it should be in the group by statement.

3. Type-3: SELECT department, employee_name, SUM(salary) FROM employees GROUP BY department;

   Type-3 doesn't work as Error: employee_name is not in GROUP BY or aggregate.

## Window Functions
1. **Row_number**: It never skips a number, even if there is a tie. (1,2,3)
2. **RANK**: This function gives tied rows the same rank, but then skips the next rank number. (1,1,3) - Skipped 2
3. **DENSE_RANK**: This function gives tied rows the same rank, but it does not skip the next rank number. (1,1,2) - No skip 2

#### Syntax: <function_name>() OVER (PARTITION BY column_name ORDER BY column_name)
  
  -- **Partition By**: Divide the result set into smaller groups (partitions), and then apply the window function independently inside each group.
  
  -- **function_name**: any aggregate (sum, count, min, max, avg) or analytic function (row_number, lag, lead, rank) that works “over a window” of rows.

### Example for aggregate function using window function:
SELECT
    Employee,
    Department,
    Salary,
    AVG(Salary) OVER (PARTITION BY Department) AS Avg_Dept_Salary
FROM
    employees;


| Employee | Department  |  Salary | Avg_Dept_Salary |
| -------- | ----------- | ------: | ---------------:|
| Alice    | Marketing   |  60000  |         70000   |
| Bob      | Marketing   |  80000  |          70000  |
| Charlie  | Engineering | 120000  |        110000   |
| David    | Engineering | 100000  |         110000  |


**Case 1: With PARTITION BY department**

SELECT 
  name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept
FROM employees;

| name    | department | salary | rank_in_dept |
| ------- | ---------- | -----: | -----------: |
| Alice   | HR         |  60000 |            1 |
| Bob     | HR         |  55000 |            2 |
| Charlie | IT         |  80000 |            1 |
| David   | IT         |  78000 |            2 |
| Emma    | Sales      |  72000 |            1 |
| Frank   | Sales      |  71000 |            2 |


**Case 2: Without PARTITION BY**

SELECT 
  name, department, salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS overall_rank
FROM employees;

| name    | department | salary | overall_rank |
| ------- | ---------- | -----: | -----------: |
| Charlie | IT         |  80000 |            1 |
| David   | IT         |  78000 |            2 |
| Emma    | Sales      |  72000 |            3 |
| Frank   | Sales      |  71000 |            4 |
| Alice   | HR         |  60000 |            5 |
| Bob     | HR         |  55000 |            6 |

## Ntile:
#### Synatx: NTILE (n) OVER (PARTITION BY column ORDER BY column)
1. **n** → Number of groups/buckets you want.
2. **PARTITION BY** → Optional, divides data into groups before applying NTILE.
3. **ORDER BY** → Required; defines how the rows are distributed into buckets.




## Lag and Lead:

## Joins


## Subqueries


## CTE


## Common Questions in Interview


