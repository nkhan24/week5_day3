# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks


### CTEs

1) Calculate the average salary of all employees

```sql
SELECT first_name,last_name,salary, AVG(salary) OVER (ORDER BY salary) AS avg_salary
FROM employees
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
SELECT departments.id, departments.name, AVG(employees.salary) AS department_avg FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
WITH departments_avgs (id, name, departments_avgs) AS
(
SELECT departments.id, departments.name, AVG(employees.salary) AS department_avg FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
)
SELECT
employees.first_name,
employees.last_name,
employees.salary,
departments_avgs.name,
departments_avgs.departments_avg,
employees.salary/departments_avgs.department_avg :: FLOAT
FROM employees
INNER JOIN departments_avgs
ON employees.department_id = departments_avgs.id
```

4) Find the employee with the highest ratio in Argentina

```sql
WITH departments_avgs (id, name, departments_avgs) AS
(
SELECT departments.id, departments.name, AVG(employees.salary) AS department_avg FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
)
SELECT
employees.first_name,
employees.last_name,
employees.salary,
departments_avgs.name,
departments_avgs.departments_avg,
employees.salary/departments_avgs.department_avg :: FLOAT
FROM employees
INNER JOIN departments_avgs
ON employees.department_id = departments_avgs.id
ORDER BY ratio DESC
LIMIT 1
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
<!--Copy solution here-->
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
SELECT
first_name,
last_name,
salary,
start_date,
SUM(salary) OVER(ORDER BY start_date DESC) AS run_tot
FROM employees
```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
SELECT
first_name,
last_name,
salary,
start_date,
RANK () OVER(ORDER BY start_date DESC) AS ranking_1
DENSE_RANK () OVER (ORDER BY start_date DESC) AS ranking_2
FROM employees
```

3) Find how many employees there are from each country

```sql
SELECT
DISTINCT country,
COUNT (*) OVER (PARTITION BY country)
FROM employees
```

4) Show how the average salary cost for each department has changed as the number of employees has increased

```sql
SELECT 
employees.first_name,
employees.last_name,
employees.salary,
employees.start_date,
departments.name
AVG(employees.salary) OVER (PARTITION BY employees.department_id ORDER BY employees.start_date DESC)
FROM employees
INNER JOIN departments ON employees.department_id = departments.id
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
<!--Copy solution here-->
```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
SELECT
MAX(employees.salary) AS maxim
MIN(employees.salary) AS minim
FROM employees
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
SELECT
salary,
first_name,
last_name,
(MAX(employees.salary)OVER ())-salary AS maxim_diff,
salary - (MIN(employees.salary)OVER ()) AS minim_diff
FROM employees
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
<!--Copy solution here-->
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
<!--Copy solution here-->
```

