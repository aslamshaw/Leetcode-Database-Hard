# LeetCode Problem 3451: Find Invalid IP Addresses

## Problem Description

You are given a table that logs server requests, containing IP addresses and HTTP status codes. The goal is to identify **invalid IPv4 addresses** based on specific criteria.

An IPv4 address is considered **invalid** if:
- It contains **more than 4 octets**, or **fewer than 4 octets** (i.e., not separated into four parts by three dots).
- Any octet has **a value greater than 255**.
- Any octet has **leading zeros** (e.g., `01`, `002`).

Your task is to return a list of these invalid IPs along with the number of times each appears in the log. The final output should be sorted by:
1. `invalid_count` in descending order
2. `ip` in descending order (alphabetically)

### Tables:

#### logs Table:

| log_id | ip              | status_code |
|--------|------------------|-------------|
| 1      | 172.16.300.2     | 404         |
| 2      | 172.016.0.1      | 200         |
| 3      | 172.16.0.1       | 200         |
| 4      | 172.16           | 500         |
| 5      | 172.016.0.1      | 200         |
| 6      | 172.16.300.2     | 404         |
| 7      | 172.16           | 200         |

### Expected Output:

| ip              | invalid_count |
|------------------|----------------|
| 172.16.300.2     | 2              |
| 172.016.0.1      | 2              |
| 172.16           | 2              |

### Problem Constraints:
- IPs are in standard dotted-decimal format.
- Each log entry contains a single IP string.
- Assume all IPs are non-null.
- Use strict IPv4 validation (i.e., reject partial matches or padded values).

---

## Solution Approaches

### Approach 1: Regex-Based Validation

This method uses a carefully constructed **regular expression (regex)** to determine whether each IP address strictly matches the definition of a valid IPv4 address.

#### SQL Query:
```sql
SELECT ip, COUNT(*) AS invalid_count  
FROM logs  
WHERE NOT REGEXP_LIKE(ip, '^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$')  
GROUP BY ip  
ORDER BY invalid_count DESC, ip DESC;
```

#### Explanation:
- The regex matches exactly four numeric octets separated by dots.
- Each octet must be between `0` and `255`, and **must not** have leading zeros unless it's `0` alone.
- The `^` (start of string) and `$` (end of string) anchors ensure the regex checks the entire string — not just a valid part embedded in a larger invalid one.
- IPs that **do not** match this pattern are counted as invalid.
- The query groups by IP and counts how often each invalid IP appears.
- Finally, it orders the results by frequency (`invalid_count`) and IP address in descending order.

---

#### Why `\b((...)\.){3}(...)\b` Doesn’t Work for Strict IPv4 Validation:

- `\b` is a **word boundary** anchor, not a full-string matcher.
- While it helps separate an IP from letters/digits, it still allows **partial matches** within a longer string.

For example, this pattern:
```regex
((\d{1,3})\.){3}\d{1,3}
```
would match:
- `"abc172.16.0.1xyz"` ✅ (partial match inside a string)
- `"172.016.0.1"` ✅ (technically matches pattern, but `016` has leading zero)

With `^` and `$`:
```regex
^((\d{1,3})\.){3}\d{1,3}$
```
You ensure:
- `"172.16.0.1"` ✅ valid
- `"172.16.0.1abc"` ❌ invalid (extra characters at the end)
- `"abc172.16.0.1"` ❌ invalid (extra characters at the start)
- `"172.016.0.1"` ❌ invalid (leading zeros)

---

#### Why Leading Zeros Might Slip Through:

Consider a loose octet regex like:
```regex
([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])
```

When applied to `001`:
1. Regex engine matches `'0'` first (valid single digit).
2. Then it keeps parsing `'01'` (also valid under parts of the pattern).
3. So `'001'` is incorrectly accepted — it's treated as either multiple matches or one relaxed match.

**Lesson**: If you don’t explicitly disallow leading zeros, your regex may incorrectly validate them due to how regex engines parse left-to-right.

---

## Performance Analysis

### Method 1 (Regex-Based Validation):

- **Time complexity**: O(n), where n is the number of rows in the `logs` table. Each IP is checked once via regex.
- **Space complexity**: O(k), where k is the number of distinct invalid IPs stored in memory during grouping.

---

## Conclusion

- **Regex-based validation** is concise and effective for this problem.
- It provides a declarative way to validate complex IP address rules in SQL.
- Using start (`^`) and end (`$`) anchors, and ensuring strict matching, makes it reliable for production use.
