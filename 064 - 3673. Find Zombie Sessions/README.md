# LeetCode Problem 3673: Find Zombie Sessions

## Problem Description

You are given a table of app events for multiple users. Each event belongs to a session identified by `session_id` and includes an event type, timestamp, and optionally an event value. Your task is to detect **zombie sessions**, which are sessions where users appear active but exhibit abnormal behavior patterns.  

A session qualifies as a zombie if **all** of the following hold:

- The session lasts more than 30 minutes.  
- The session includes at least 5 scroll events.  
- The ratio of click events to scroll events is below 0.20.  
- No purchases occur in the session.  

Return the list of zombie sessions with their `session_id`, `user_id`, total session duration in minutes, and scroll count. Sort results first by `scroll_count` descending, then by `session_id` ascending.

### Tables:

#### app_events Table:

| event_id | user_id | event_timestamp     | event_type | session_id | event_value |
|----------|---------|-------------------|------------|------------|-------------|
| 101      | 301     | 2024-05-01 09:00:00 | app_open   | X001       | NULL        |
| 102      | 301     | 2024-05-01 09:05:00 | scroll     | X001       | 450         |
| 103      | 301     | 2024-05-01 09:12:00 | scroll     | X001       | 700         |
| 104      | 301     | 2024-05-01 09:20:00 | scroll     | X001       | 600         |
| 105      | 301     | 2024-05-01 09:25:00 | scroll     | X001       | 550         |
| 106      | 301     | 2024-05-01 09:35:00 | scroll     | X001       | 650         |
| 107      | 301     | 2024-05-01 09:40:00 | app_close  | X001       | NULL        |
| 108      | 302     | 2024-05-01 10:00:00 | app_open   | X002       | NULL        |
| 109      | 302     | 2024-05-01 10:05:00 | click      | X002       | NULL        |
| 110      | 302     | 2024-05-01 10:08:00 | scroll     | X002       | 300         |
| 111      | 302     | 2024-05-01 10:10:00 | purchase   | X002       | 45          |
| 112      | 303     | 2024-05-01 11:00:00 | app_open   | X003       | NULL        |
| 113      | 303     | 2024-05-01 11:15:00 | scroll     | X003       | 900         |
| 114      | 303     | 2024-05-01 11:25:00 | scroll     | X003       | 850         |
| 115      | 303     | 2024-05-01 11:30:00 | click      | X003       | NULL        |
| 116      | 303     | 2024-05-01 11:35:00 | scroll     | X003       | 750         |
| 117      | 304     | 2024-05-01 12:00:00 | app_open   | X004       | NULL        |
| 118      | 304     | 2024-05-01 12:03:00 | scroll     | X004       | 400         |
| 119      | 304     | 2024-05-01 12:06:00 | click      | X004       | NULL        |

### Expected Output:

| session_id | user_id | session_duration_minutes | scroll_count |
|------------|---------|--------------------------|--------------|
| X001       | 301     | 40                       | 5            |

### Problem Constraints:
- The session duration is calculated from the first event to the last event of the session.  
- Only sessions with at least 5 scroll events are considered.  
- Click-to-scroll ratio must be strictly less than 0.20.  
- Sessions with any purchase events are excluded.  

---

## Solution Approaches

### Approach 1: Aggregate and Filter

This approach aggregates events by `session_id` and `user_id`, computes session duration and counts of scroll/click/purchase events, then filters sessions based on the zombie criteria.

#### SQL Query:
```sql
SELECT session_id, user_id, 
TIMESTAMPDIFF(MINUTE, MIN(event_timestamp), MAX(event_timestamp)) AS session_duration_minutes, 
SUM(event_type = 'scroll') AS scroll_count
FROM app_events 
GROUP BY user_id, session_id
HAVING session_duration_minutes > 30
AND scroll_count >= 5
AND SUM(event_type = 'click') / scroll_count < 0.2
AND SUM(event_type = 'purchase') = 0
ORDER BY scroll_count DESC, session_id;
```

#### Explanation:
- Group events by `session_id` and `user_id`.  
- Compute session duration as the difference in minutes between the earliest and latest event.  
- Count the number of scroll events.  
- Filter sessions where:  
  - duration > 30 minutes  
  - scroll_count ≥ 5  
  - click_count / scroll_count < 0.20  
  - purchase_count = 0  
- Order results by `scroll_count` descending, then `session_id` ascending.

---

## Performance Analysis

### Method 1 (Aggregate and Filter):

- **Time complexity**: O(n), where n is the number of events (single scan for aggregation).  
- **Space complexity**: O(m), where m is the number of unique `(user_id, session_id)` pairs.  

---

## Conclusion

- **Aggregate and Filter**: Efficient for moderate to large datasets, simple logic using grouping and conditional aggregation. Limited by in-memory aggregation if the dataset is extremely large.  

### Final Conclusion:
- The aggregate-and-filter approach is optimal for this problem, providing a straightforward and readable solution while efficiently filtering zombie sessions.
