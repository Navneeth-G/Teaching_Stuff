# Event Pairing in SQL 📝

## How to Recognize
- Table has pairs of events in separate rows (start/end timestamps)
- Need to match these pairs to calculate something (like duration)
- Keywords: start/end, in/out, before/after

## Two Ways to Solve

### 1. SELF JOIN Method 🤝
```sql
SELECT s.id, (e.time - s.time) as duration
FROM events s 
JOIN events e 
  ON s.id = e.id 
  AND s.type = 'start' 
  AND e.type = 'end'
```
Think: "Match each start with its end"

### 2. Aggregation Method 📊
```sql
SELECT id,
    MAX(CASE WHEN type = 'end' THEN time END) -
    MIN(CASE WHEN type = 'start' THEN time END) as duration
FROM events
GROUP BY id
```
Think: "Group events and pick start/end times"

# When to Use Which Method

### 1. Use SELF JOIN When:
```sql
-- If you need details from both start AND end events
SELECT 
    s.id,
    s.user_name,      -- start event details
    s.location_start, -- start event details
    e.location_end,   -- end event details
    e.battery_level,  -- end event details
    (e.time - s.time) as duration
FROM events s 
JOIN events e 
    ON s.id = e.id 
    AND s.type = 'start' 
    AND e.type = 'end'
```
Think: "I need info from BOTH start and end rows"

### 2. Use Aggregation When:
```sql
-- If you just need the time difference or simple calculations
SELECT 
    id,
    MAX(CASE WHEN type = 'end' THEN time END) -
    MIN(CASE WHEN type = 'start' THEN time END) as duration
FROM events
GROUP BY id
```
Think: "I just need to calculate differences"

### Real Example:
1. Ride-sharing app:
   - JOIN Method: Need rider's pickup location AND dropoff location
   - Aggregate Method: Just need to calculate ride duration

2. Factory process:
   - JOIN Method: Need operator name at start AND error codes at end

## Real-World Examples
1. Process Monitoring
   - Track process start/end times
   - Calculate process duration

2. User Sessions
   - Track login/logout
   - Calculate session length

3. Delivery Tracking
   - Track pickup/delivery
   - Calculate transit time

## Common Gotchas
- Ensure proper join conditions
- Handle NULL values appropriately
- Consider data type casting
- Watch for duplicate events

The key is:
- Need multiple columns from both events? → Use JOIN
- Just need time difference? → Use Aggregate

-------------------------------------------------------------------------------------------------------------
## Sample data

### Query Used

```sql
SELECT 
    id,
    MAX(CASE WHEN type = 'end' THEN time END) -
    MIN(CASE WHEN type = 'start' THEN time END) as duration
FROM events
GROUP BY id;
```

This query calculates the duration between the `start` and `end` times for each `id`.

---

## Step 1: Sample Data

| id  | type  | time |
|-----|-------|------|
| 1   | start | 10   |
| 1   | end   | 20   |
| 2   | start | 15   |
| 2   | end   | 35   |
| 3   | start | 50   |
| 3   | end   | 80   |

---

## **Step 2: Using `CASE WHEN` to Extract Start and End Times**

The query uses `CASE WHEN` to **filter out start and end times** separately.

```sql
SELECT 
    id,
    CASE WHEN type = 'start' THEN time END AS start_time,
    CASE WHEN type = 'end' THEN time END AS end_time
FROM events;
```

### **Extracted Data Table:**
| id  | `start_time` | `end_time` |
|-----|-------------|-----------|
| 1   | 10          | NULL      |
| 1   | NULL        | 20        |
| 2   | 15          | NULL      |
| 2   | NULL        | 35        |
| 3   | 50          | NULL      |
| 3   | NULL        | 80        |

---

## **Step 3: Applying Aggregations (`MIN` and `MAX`)**
Now, we use **MIN and MAX** to group the data and extract start and end times per `id`.

```sql
SELECT 
    id,
    MIN(CASE WHEN type = 'start' THEN time END) AS min_start_time,
    MAX(CASE WHEN type = 'end' THEN time END) AS max_end_time
FROM events
GROUP BY id;
```

### **Aggregated Data Table:**
| id  | MIN(start_time) | MAX(end_time) |
|-----|---------------|-------------|
| 1   | 10            | 20          |
| 2   | 15            | 35          |
| 3   | 50            | 80          |

---

## **Step 4: Calculating the Duration**
The final calculation subtracts `start_time` from `end_time`.

```sql
SELECT 
    id,
    MAX(CASE WHEN type = 'end' THEN time END) -
    MIN(CASE WHEN type = 'start' THEN time END) AS duration
FROM events
GROUP BY id;
```

### **Final Output:**
| id  | duration |
|-----|----------|
| 1   | 10       |
| 2   | 20       |
| 3   | 30       |

---

## **Conclusion**
This SQL query efficiently calculates the **time difference** between `start` and `end` events for each `id`.

