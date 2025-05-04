### Reversing Values in a Column Based on Custom Sorting Logic (SQL)
### 🧩 **Problem Statement**

Given a table with ProductID, Product, and Category, 
we need to reassign ProductID values within each category, such that:

- The new ProductIDs are the reverse of the original ProductIDs (within that category),
- The rows themselves (Product name + Category) remain unchanged.


**Input Table Example:**

| ProductID | Product    | Category    |
|-----------|-----------|-------------|
| 1         | Laptop     | Electronics |
| 2         | Smartphone | Electronics |
| 3         | Tablet     | Electronics |
| 9         | Printer    | Electronics |
| 4         | Headphones | Accessories |
| 5         | Smartwatch | Accessories |
| 6         | Keyboard   | Accessories |
| 7         | Mouse      | Accessories |
| 8         | Monitor    | Accessories |


**Expected Output Example:**

| ProductID | Product    | Category    |
|-----------|-----------|-------------|
| 9         | Laptop     | Electronics |
| 3         | Smartphone | Electronics |
| 2         | Tablet     | Electronics |
| 1         | Printer    | Electronics |
| 8         | Headphones | Accessories |
| 7         | Smartwatch | Accessories |
| 6         | Keyboard   | Accessories |
| 5         | Mouse      | Accessories |
| 4         | Monitor    | Accessories |


---

### 💡 **Hints Before Jumping In**


- You're not just sorting rows.
- You're reassigning column values — this requires mapping.
- The reassignment is **relative to the position** within a group.


---

### 🧠 **Thought Process & Step-by-Step Reasoning**


#### Step 1: What’s Really Changing?
We’re not sorting rows. We’re assigning new values to `ProductID`.

#### Step 2: What’s the Partition Group?
Each category needs to be treated independently → use `PARTITION BY Category`.

#### Step 3: What’s the Rule?
Sort all ProductIDs in ascending order, and assign the reversed values back as NewProductID.

#### Step 4: How to Assign Position?
Use `ROW_NUMBER()`:
- Once for ascending order
- Once for descending order

#### Step 5: Join Those Two Together
This lets us map each position from ascending to its counterpart in descending.


### 🔄 **SQL Solution**

```sql
WITH ranked_products AS (
  SELECT 
    ProductID,
    Product,
    Category,
    ROW_NUMBER() OVER (PARTITION BY Category ORDER BY ProductID ASC) AS rn
  FROM products
),
reversed_ids AS (
  SELECT 
    ProductID AS reversed_id,
    Category,
    ROW_NUMBER() OVER (PARTITION BY Category ORDER BY ProductID DESC) AS rn
  FROM products
)
SELECT 
  r.ProductID AS Original_ProductID,
  r.Product,
  r.Category,
  rev.reversed_id AS NewProductID
FROM ranked_products r
JOIN reversed_ids rev
  ON r.Category = rev.Category AND r.rn = rev.rn
ORDER BY NewProductID DESC;
```

---

### 🧪 **Table Transformations at Each Step**

You can paste in screenshots or markdown tables from earlier like:

#### ➤ ranked\_products (row number ascending):

| ProductID | Product    | Category    |   rn |
|-----------|-----------|-------------|------|
| 1         | Laptop     | Electronics |    1 |
| 2         | Smartphone | Electronics |    2 |
| 3         | Tablet     | Electronics |    3 |
| 9         | Printer    | Electronics |    4 |
| 4         | Headphones | Accessories |    1 |
| 5         | Smartwatch | Accessories |    2 |
| 6         | Keyboard   | Accessories |    3 |
| 7         | Mouse      | Accessories |    4 |
| 8         | Monitor    | Accessories |    5 |


#### ➤ reversed\_ids (row number descending):

| reversed_id | Product    | Category    |   rn |
|-------------|-----------|-------------|------|
| 1           | Laptop     | Electronics |    4 |
| 2           | Smartphone | Electronics |    3 |
| 3           | Tablet     | Electronics |    2 |
| 9           | Printer    | Electronics |    1 |
| 4           | Headphones | Accessories |    5 |
| 5           | Smartwatch | Accessories |    4 |
| 6           | Keyboard   | Accessories |    3 |
| 7           | Mouse      | Accessories |    2 |
| 8           | Monitor    | Accessories |    1 |


#### ➤ Final joined result:

| Original_ProductID | Product    | Category    | rn | rn_of_rev | Original_ProductID_of_rev | Product_of_rev | Category_of_rev | NewProductID |
|--------------------|-----------|-------------|----|-----------|----------------------------|----------------|------------------|--------------|
| 1                  | Laptop     | Electronics | 1  | 1         | 9                          | Printer        | Electronics       | 9            |
| 2                  | Smartphone | Electronics | 2  | 2         | 3                          | Tablet         | Electronics       | 3            |
| 3                  | Tablet     | Electronics | 3  | 3         | 2                          | Smartphone     | Electronics       | 2            |
| 9                  | Printer    | Electronics | 4  | 4         | 1                          | Laptop         | Electronics       | 1            |
| 4                  | Headphones | Accessories | 1  | 1         | 8                          | Monitor        | Accessories       | 8            |
| 5                  | Smartwatch | Accessories | 2  | 2         | 7                          | Mouse          | Accessories       | 7            |
| 6                  | Keyboard   | Accessories | 3  | 3         | 6                          | Keyboard       | Accessories       | 6            |
| 7                  | Mouse      | Accessories | 4  | 4         | 5                          | Smartwatch     | Accessories       | 5            |
| 8                  | Monitor    | Accessories | 5  | 5         | 4                          | Headphones     | Accessories       | 4            |


---

### **Key Takeaways**
- Use ROW_NUMBER to anchor order within a group.
- Self-join on row_number to assign values from a reversed order.
- Understand the difference between sorting rows and reassigning values.

--- 

##  **Practice Problem Statement: Reassigning Employee Levels**

### Task:

> For each department, reassign the `EmpID` values such that the employee with the **lowest original `EmpID`** receives the **highest available `EmpID`** *within that same department*, and so on.

###  Table: `employees`

| EmpID | Name    | Department  |
| ----- | ------- | ----------- |
| 101   | Alice   | Engineering |
| 102   | Bob     | Engineering |
| 103   | Charlie | Engineering |
| 201   | Diana   | HR          |
| 202   | Ethan   | HR          |
| 301   | Frank   | Sales       |
| 302   | Grace   | Sales       |
| 303   | Helen   | Sales       |
| 304   | Ian     | Sales       |

---



###  Expected Output (for example):

| Original\_EmpID | Name    | Department  | New\_EmpID |
| --------------- | ------- | ----------- | ---------- |
| 101             | Alice   | Engineering | 103        |
| 102             | Bob     | Engineering | 102        |
| 103             | Charlie | Engineering | 101        |
| ...             | ...     | ...         | ...        |

---



