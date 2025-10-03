SQL Problem: Find the Second Highest Distinct Salary
This document outlines the problem and provides the optimized SQL solution to find the second highest distinct salary from the Employee table, ensuring that the result is a single-row, single-column output that returns NULL if the salary does not exist.

Table Schema
Column Name

Type

id

int

salary

int

id is the primary key.

Examples
Example 1: Second Highest Salary Exists

id

salary

1

100

2

200

3

300

Output:

SecondHighestSalary

200

Example 2: No Second Highest Salary

id

salary

1

100

Output:

SecondHighestSalary

null

Optimal SQL Solution (Using DENSE_RANK() with Scalar Subquery)
This solution uses DENSE_RANK() to rank distinct salaries and wraps the result in a scalar subquery with an aggregate function (MAX) to guarantee a single result row, even when multiple employees share the second highest salary.

SELECT
    (SELECT MAX(salary)
     FROM (
         -- Step 1: Rank all salaries in descending order
         SELECT
             salary,
             -- DENSE_RANK assigns the same rank to ties (duplicate salaries)
             -- and ensures the next rank is consecutive.
             DENSE_RANK() OVER (ORDER BY salary DESC) as emp_rank 
         FROM
             Employee
         ) AS e
     -- Step 2: Filter for the second rank
     WHERE
         e.emp_rank = 2
    ) AS SecondHighestSalary;

Key Logic Points:
DENSE_RANK(): Correctly handles duplicate salary entries (e.g., three people earning $300 are all rank 1), ensuring the next highest unique salary gets rank 2.

WHERE e.emp_rank = 2: Isolates the second highest distinct salary value(s).

SELECT MAX(salary): When multiple employees share the second highest salary (e.g., two people earn $200), the MAX() function collapses these identical values into a single result, preventing the inner query from returning multiple rows.

Outer Scalar Subquery: SELECT ( ... ) AS SecondHighestSalary guarantees a single final output row. If the inner query returns no rows (no second salary), the final output is automatically set to NULL.
