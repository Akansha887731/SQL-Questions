# Second Highest Distinct Salary 💰

This problem requires finding the second highest *distinct* salary from the `Employee` table. If a second highest salary does not exist (e.g., the table has fewer than two unique salaries), the solution should return `null`.

## 📝 Problem Description

Find the second highest distinct salary from the `Employee` table. If there is no second highest distinct salary, return `null`. The result should be aliased as `SecondHighestSalary`.

### 📋 Examples
Example 1: Second Highest Salary Exists
Input (Employee Table):
| id | salary |
| :--| :----- |
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |

Output:
| SecondHighestSalary |
| :------------------ |
| 200                 |

Example 2: No Second Highest Salary
Input (Employee Table):
| id | salary |
| :--| :----- |
| 1  | 100    |

Output:
| SecondHighestSalary |
| :------------------ |
| null                |

---

## 🏗️ Table Schema

**Table:** `Employee`

| Column Name | Type | Description |
| :---------- | :--- | :---------- |
| `id`        | `int`| Primary key for this table. |
| `salary`    | `int`| The salary of the employee. |

---

## 💡 SQL Solution using Window Functions

This solution uses the `DENSE_RANK()` window function to assign a rank to each distinct salary, ordered in descending order. Then, it selects the salary corresponding to the rank of 2.

```sql
SELECT
    (
        SELECT
            e.salary
        FROM
            (
                -- 1. Use DENSE_RANK() to assign a rank to each distinct salary.
                -- DENSE_RANK handles ties and gaps correctly for finding the 'Nth' distinct value.
                SELECT
                    salary,
                    DENSE_RANK() OVER (ORDER BY salary DESC) as emp_rank
                FROM
                    Employee
            ) AS e
        -- 2. Filter for the second-highest rank.
        WHERE
            e.emp_rank = 2
        -- 3. In case the subquery returns multiple rows (which DENSE_RANK prevents for the *distinct* second salary,
        -- but good practice for ensuring a single result), LIMIT is used.
        LIMIT 1
    ) AS SecondHighestSalary;
```

### 🔍 Alternative Solution using LIMIT and OFFSET
This approach avoids window functions and is often simpler and faster in MySQL. It finds the second distinct value by ordering all distinct salaries descendingly, skipping the first one (OFFSET 1), and taking the next one (LIMIT 1). The entire statement is wrapped to ensure NULL is returned if no row exists.


```sql
SELECT
    (
        SELECT DISTINCT
            salary
        FROM
            Employee
        ORDER BY
            salary DESC
        -- Skip the highest salary (rank 1)
        LIMIT 1
        -- Start from the second highest distinct salary (rank 2)
        OFFSET 1
    ) AS SecondHighestSalary;
```
