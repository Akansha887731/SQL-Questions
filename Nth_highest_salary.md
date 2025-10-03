# Nth Highest Salary 💰

## 📝 Problem Description

Write a SQL function named `getNthHighestSalary(N)` to find the **Nth highest distinct salary** from the `Employee` table.

If there are less than $N$ distinct salaries, the function should return `NULL`. The result must be aliased as the function call itself (e.g., `getNthHighestSalary(2)`).

---

## 🏗️ Table Schema

**Table:** `Employee`

| Column Name | Type | Description |
| :---------- | :--- | :---------- |
| `id`        | `int`| Primary key. |
| `salary`    | `int`| The employee's salary. |

---
## 📊 Examples
### Example 1: n = 2 (Second Highest Salary)
Input (Employee Table):
| id | salary |
| :--| :----- |
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |

Output:
| getNthHighestSalary(2) |
| :--------------------- |
| 200                    |

### Example 2: n = 2 (Insufficient Salaries)
Input (Employee Table):
| id | salary |
| :--| :----- |
| 1  | 100    |

Output:
| getNthHighestSalary(2) |
| :--------------------- |
| null                   |

## 💡 Solution

Two robust methods are provided below. The `LIMIT`/`OFFSET` method is often preferred in MySQL for simplicity and performance.

### Method 1: Using `LIMIT` and `OFFSET` (Recommended for MySQL)

This method orders all distinct salaries descendingly and uses the `$N - 1$` value for the `OFFSET` to skip the first $N-1$ highest salaries.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  -- Declare a variable to hold the offset value, as OFFSET requires a non-negative constant.
  -- This handles the need for N-1 cleanly.
  SET @offset_val = N - 1;

  RETURN (
    SELECT DISTINCT
        e.salary
    FROM
        Employee e
    ORDER BY
        e.salary DESC
    -- LIMIT 1 returns the single Nth highest salary.
    -- OFFSET @offset_val skips the top N-1 salaries.
    LIMIT 1 OFFSET @offset_val
  );
END
```

### Method 2: Using the DENSE_RANK() Window Function
This method is highly portable across different SQL dialects (like PostgreSQL, SQL Server, etc.) and explicitly handles the "distinct" requirement by ranking the salaries.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
    SELECT 
        (
            SELECT 
                temp.salary
            FROM
                (
                    -- 1. Use DENSE_RANK() to assign a rank to each distinct salary.
                    SELECT 
                        salary, 
                        DENSE_RANK() OVER (ORDER BY salary DESC) as sal_rank
                    FROM 
                        Employee 
                ) AS temp
            -- 2. Filter for the salary that matches the input rank N.
            WHERE 
                temp.sal_rank = N
            -- 3. LIMIT 1 ensures a single scalar result is returned.
            LIMIT 1
        )
    AS getNthHighestSalary
  );
END
```
