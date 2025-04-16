# LeetCode Problem 2793: Status of Flight Tickets

## Problem Description

You are given two tables: `Flights` and `Passengers`.

- The `Flights` table contains flight IDs and the number of available seats (capacity) for each flight.
- The `Passengers` table holds records of passengers who booked tickets, including which flight they booked and at what time.

A passenger's ticket status depends on how early they booked and how many seats are available on the flight:
- If the passenger booked early enough to secure a seat (within the flight’s capacity), their status is **Confirmed**.
- Otherwise, if the flight is already fully booked when the passenger tries to book, they are placed on the **Waitlist**.

Determine the current status ("Confirmed" or "Waitlist") for each passenger based on their booking time and the flight's capacity.

Return the result ordered by `passenger_id` in ascending order.

---

### Tables:

#### Flights Table:

| flight_id | capacity |
|-----------|----------|
| 10        | 3        |
| 20        | 1        |
| 30        | 2        |

#### Passengers Table:

| passenger_id | flight_id | booking_time        |
|--------------|-----------|---------------------|
| 501          | 10        | 2023-07-15 10:20:00 |
| 502          | 10        | 2023-07-15 09:45:00 |
| 503          | 10        | 2023-07-15 12:00:00 |
| 504          | 10        | 2023-07-15 13:15:00 |
| 505          | 20        | 2023-07-14 08:00:00 |
| 506          | 20        | 2023-07-14 09:30:00 |
| 507          | 30        | 2023-07-13 11:11:00 |
| 508          | 30        | 2023-07-13 11:12:00 |
| 509          | 30        | 2023-07-13 11:13:00 |

### Expected Output:

| passenger_id | Status    |
|--------------|-----------|
| 501          | Confirmed |
| 502          | Confirmed |
| 503          | Confirmed |
| 504          | Waitlist  |
| 505          | Confirmed |
| 506          | Waitlist  |
| 507          | Confirmed |
| 508          | Confirmed |
| 509          | Waitlist  |

### Problem Constraints:
- Each `flight_id` is unique.
- Each `passenger_id` and `booking_time` are distinct.
- A passenger is placed on the waitlist if they book after the number of confirmed bookings has already reached the flight's capacity.

---

## Solution Approaches

### Approach 1: Window Function with Cumulative Count

To determine each passenger's status, this approach uses a **window function** to count bookings for each flight in order of booking time. The logic:
- Join the `Passengers` and `Flights` tables on `flight_id`.
- For each passenger, assign a rank based on how early they booked using a `COUNT()` window function.
- Compare the count to the flight’s capacity:
  - If the cumulative count is **within** the flight’s capacity, the status is **Confirmed**.
  - Otherwise, the passenger is **Waitlisted**.

#### SQL Query:
```sql
WITH PassengerCount AS (
    SELECT *, 
           COUNT(passenger_id) OVER (PARTITION BY flight_id ORDER BY booking_time) AS cnt
    FROM Passengers
    JOIN Flights USING (flight_id)
)

SELECT passenger_id, 
       CASE 
           WHEN capacity >= cnt THEN 'Confirmed' 
           ELSE 'Waitlist' 
       END AS Status 
FROM PassengerCount;

```

#### Explanation:
- The `COUNT()` window function computes the number of passengers who booked **before or at the same time** for a given flight.
- By comparing this count with the flight's `capacity`, we determine who makes the cutoff.
- Finally, the result is selected and ordered by `passenger_id`.

---

## Performance Analysis

### Method 1 (Window Function with Cumulative Count):

- **Time complexity**: `O(n log n)` — due to sorting of passengers by `booking_time` per flight.
- **Space complexity**: `O(n)` — to store intermediate windowed results.

---

## Conclusion

- **Method 1**: Efficient and readable. Makes good use of SQL’s window functions to solve the problem in a single pass. Scales well as long as the number of passengers per flight isn't extremely large.

### Final Conclusion:
- The window function approach is effective and concise, providing an optimal solution for determining ticket statuses based on booking order and flight capacity.
