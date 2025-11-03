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

**Example: Divide Employees into 4 Salary Quartiles**

SELECT name, department_id, salary, NTILE(4) OVER (ORDER BY salary DESC) AS quartile FROM employees;

| name    | department_id | salary | quartile |
| ------- | ------------- | ------ | -------- |
| Frank   | 2             | 90000  | 1        |
| Grace   | 2             | 85000  | 1        |
| Alice   | 1             | 80000  | 2        |
| Helen   | 2             | 75000  | 2        |
| Bob     | 1             | 70000  | 2        |
| Ian     | 2             | 65000  | 3        |
| Charlie | 1             | 60000  | 3        |
| Jack    | 2             | 55000  | 3        |
| David   | 1             | 50000  | 4        |
| Eva     | 1             | 40000  | 4        |


## Lag and Lead:

LAG(column, offset, default) allows you to look back to a previous row in the same result set without using a self-join.

LEAD(column, offset, default) is the opposite of LAG(). It looks ahead to the next row in the same result set.

**column:** The column you want to look backward (LAG) or forward (LEAD) 

**offset:** How many rows back or ahead to look (default = 1)

**default:** Value to return if there is no previous or next row (instead of NULL)

**Example 1:**

SELECT name, salary, LAG(salary, 1) OVER (ORDER BY salary DESC) AS prev_salary, salary - LAG(salary, 1) OVER (ORDER BY salary DESC) AS diff_from_prev
FROM employees
WHERE department_id = 1;

| name    | salary | prev_salary | diff_from_prev |
| ------- | ------ | ----------- | -------------- |
| Alice   | 80000  | NULL        | NULL           |
| Bob     | 70000  | 80000       | -10000         |
| Charlie | 60000  | 70000       | -10000         |
| David   | 50000  | 60000       | -10000         |
| Eva     | 40000  | 50000       | -10000         |


**Example 2:**

SELECT name, salary, LEAD(salary, 1) OVER (ORDER BY salary DESC) AS next_salary, salary - LEAD(salary, 1) OVER (ORDER BY salary DESC) AS diff_from_next
FROM employees
WHERE department_id = 1;

| name    | salary | next_salary | diff_from_next |
| ------- | ------ | ----------- | -------------- |
| Alice   | 80000  | 70000       | 10000          |
| Bob     | 70000  | 60000       | 10000          |
| Charlie | 60000  | 50000       | 10000          |
| David   | 50000  | 40000       | 10000          |
| Eva     | 40000  | NULL        | NULL           |


## Joins

**INNER JOIN:** Returns only the rows that have a match in both tables.

**LEFT JOIN (or LEFT OUTER JOIN):** Returns all rows from the left table, and matched rows from the right. If no match, the right-side columns return NULL.

**RIGHT JOIN (or RIGHT OUTER JOIN):** Returns all rows from the right table, and matched rows from the left. If no match, left-side columns return NULL.

**FULL OUTER JOIN:** Returns all rows from both tables, with NULL where there’s no match. (NULL where no match on either side)

**CROSS JOIN:** Returns the Cartesian product of both tables — every row from one joined with every row from the other. (No condition used and it didn't give null by itself)

**SELF JOIN:** Join a table to itself (usually to compare rows).

**Example 1:**

**Table 1:**

| Row_No | Value |
| :----: | :---: |
|    1   |   1   |
|    2   |   1   |
|    3   |   1   |
|    4   |   1   |
|    5   |   8   |
|    6   |   8   |
|    7   |   8   |
|    8   |   8   |

**Table 2:**

| Row_No | Value |
| :----: | :---: |
|    1   |   1   |
|    2   |   1   |
|    3   |   1   |
|    4   |   1   |
|    5   |   8   |
|    6   |   8   |
|    7   |   8   |
|    8   |   8   |


Unique Matching: 1 and 8

Count of '1' in T1: 4

Count of '1' in T2: 4

Total Rows for '1': 4*4 = 16

Count of '8' in T1: 4

Count of '8' in T2: 4

Total Rows for '8': 4*4 = 16

Total Matches (M) = 16+16 =32 

Let $U_1$ be the number of rows in T1 that have no match in T2.

Let $U_2$ be the number of rows in T2 that have no match in T1.

$U_1$ = 0 (all rows are matching)

$U_2$ = 0 (all rows are matching)

**Formula:**

INNER JOIN: M

LEFT JOIN: M + $U_1$

RIGHT JOIN: M + $U_2$

FULL OUTER JOIN: M + $U_1$ + $U_2$

CROSS JOIN: $\text{Total T1 Rows} \times \text{Total T2 Rows}$

**Answers**

INNER JOIN: The total resulting rows is 32.

LEFT JOIN: The total resulting rows is 32.

RIGHT JOIN: The total resulting rows is 32.

FULL OUTER JOIN: The total resulting rows is 32.

CROSS JOIN: The total resulting rows is 64.


**Example 2:**

**Table 1:**

| Row_No | Value |
| :----: | :---: |
|    1   |   1   |
|    2   |   1   |
|    3   |   1   |
|    4   |   1   |
|    5   |   8   |
|    6   |   8   |
|    7   |   8   |
|    8   |   8   |
|    9   |   2   |

**Table 2:**

| Row_No | Value |
| :----: | :---: |
|    1   |   1   |
|    2   |   1   |
|    3   |   6   |
|    4   |   7   |
|    5   |   8   |
|    6   |   8   |
|    7   |   5   |
|    8   |   4   |
|    9   |   3   |

Unique Matching: 1 and 8

Count of '1' in T1: 4

Count of '1' in T2: 2

Total Rows for '1': 4*2 = 8

Count of '8' in T1: 4

Count of '8' in T2: 2

Total Rows for '8': 4*2 = 8

Total Matches (M) = 8+8 = 16

$U_1$ = 1

$U_2$ = 5

**Answers**

INNER JOIN: The total resulting rows is 16

LEFT JOIN: The total resulting rows is 17.

RIGHT JOIN: The total resulting rows is 21.

FULL OUTER JOIN: The total resulting rows is 22.

CROSS JOIN: The total resulting rows is 81.

## 1. Union and Union all

Both are used to combine results of two or more SELECT queries.

**UNION** and **UNION AL**
1. Combines rows from both queries
2. Removes duplicates automatically (like DISTINCT) in union but not in union all.
3. Both SELECT queries must return the same count of columns

**Syntax**

SELECT name, department FROM employees_2024

UNION

SELECT name, department FROM employees_2025;

## 2. INTERSECT

1. Finds common rows between two queries (like set intersection).
2. Removes duplicates automatically.

**Syntax**

SELECT name FROM employees_2024

INTERSECT

SELECT name FROM employees_2025;

**Result:** Shows names present in both years.

## 3. EXCEPT

1. Finds rows present in the first query but not in the second.

**Syntax**

SELECT name FROM employees_2024

EXCEPT

SELECT name FROM employees_2025;

**Result:** Returns name only those from 2024 not found in 2025.

## Subqueries

**Original Table**

| name    | department | salary |
| ------- | ---------- | -----: |
| Alice   | HR         |  60000 |
| Bob     | HR         |  55000 |
| Charlie | IT         |  80000 |
| David   | IT         |  78000 |
| Emma    | Sales      |  72000 |

**Example 1 — Subquery in WHERE clause**

Question: Find employees who earn more than the average salary.

SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

**O/P**

| name    | salary |
| ------- | -----: |
| Charlie |  80000 |
| David   |  78000 |
| Emma    |  72000 |


**Example 2 — Subquery with IN**

Question: Find employees who work in departments where the average salary is greater than 70000.

SELECT name, department FROM employees WHERE department IN (

  SELECT department
  FROM employees
  GROUP BY department
  HAVING AVG(salary) > 70000
);

**O/P**

| name    | department |
| ------- | ---------- |
| Charlie | IT         |
| David   | IT         |
| Emma    | Sales      |


**Example 3 — Subquery in FROM clause (Derived Table)**

Question: Find departments with an average salary above 70,000.

SELECT department, avg_salary FROM (

  SELECT department, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
) AS dept_avg

WHERE avg_salary > 70000;

**O/P**

| department | avg_salary |
| ---------- | ---------: |
| IT         |      79000 |
| Sales      |      72000 |


**Example 4 — Subquery in SELECT clause**

Question: Show each employee’s salary and how much above the company average it is.

SELECT name, salary, 

  salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg FROM employees;

**O/P**

| name    | salary | diff_from_avg |
| ------- | -----: | ------------: |
| Alice   |  60000 |         -9000 |
| Charlie |  80000 |        +11000 |

  
**Example 5 — Correlated Subquery**

A correlated subquery is a subquery that depends on the outer query for its value.

That means — the inner query runs once for every row of the outer query.

**Syntax:**

SELECT column1, column2 FROM table1 t1 WHERE column3 > (SELECT AVG(column3) FROM table1 t2

WHERE t2.department = t1.department);

-- t2.department = t1.department → this line makes it correlated, because the inner query depends on t1

-- If we remove that line t2.department = t1.department, then The inner query now calculates the average salary of all employees (not per department).

-- So the outer query will return every employee whose salary is greater than the overall average, across the entire company.


**Question: Employees earning more than their department average**

SELECT name, department, salary FROM employees e1 WHERE salary > (

  SELECT AVG(salary) FROM employees e2
  
  WHERE e2.department = e1.department
);

**O/P**

| name    | department | salary |
| ------- | ---------- | -----: |
| Alice   | HR         |  60000 |
| Charlie | IT         |  80000 |
| Emma    | Sales      |  72000 |


## CTE

**Basic Synatx:** WITH cte_name AS (
    SELECT column1, column2, ...
    FROM table_name
    WHERE condition
)

SELECT *
FROM cte_name;

**Example 1: Basic CTE — Filtering Data**

WITH high_salary AS (SELECT name, department, salary FROM employees WHERE salary > 70000
)

SELECT * FROM high_salary;

**O/P**

| name    | department | salary |
| ------- | ---------- | ------ |
| Charlie | IT         | 80000  |
| David   | IT         | 78000  |
| Emma    | Sales      | 72000  |


**Example 2: CTE with Aggregation** 

Q; Find the average salary by department

WITH dept_avg AS (SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department)

SELECT * FROM dept_avg;

**O/P**

| department | avg_salary |
| ---------- | ---------- |
| HR         | 57500      |
| IT         | 79000      |
| Sales      | 72000      |


**Example 3: CTE + Main Query Join** 

Q: Show employees who earn more than their department’s average salary.

WITH dept_avg AS (SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department)

SELECT e.name, e.department, e.salary, d.avg_salary FROM employees e

JOIN dept_avg d

ON e.department = d.department

WHERE e.salary > d.avg_salary;

**O/P**

| name    | department | salary | avg_salary |
| ------- | ---------- | ------ | ---------- |
| Charlie | IT         | 80000  | 79000      |
| Emma    | Sales      | 72000  | 72000      |



**Example 4: CTE used for Ranking**

Q: Find the top-paid employee in each department.

WITH ranked AS (SELECT name, department, salary, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept FROM employees)

SELECT name, department, salary

FROM ranked

WHERE rank_in_dept = 1;

**O/P**

| name    | department | salary |
| ------- | ---------- | ------ |
| Alice   | HR         | 60000  |
| Charlie | IT         | 80000  |
| Emma    | Sales      | 72000  |


## Common Questions in Interview




