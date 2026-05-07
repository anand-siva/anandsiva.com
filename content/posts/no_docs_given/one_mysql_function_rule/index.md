+++
title = "WITH RECURSIVE: One MySQL Function to Rule Them All"
date = "2026-04-15T20:01:20-04:00"
author = "Anand Siva"
draft = false
cover = "with_recursive_cover.png"

tags = ["mysql", "sql", "cte", "recursion", "database"]
categories = ["devops"]
publications = ["No Docs Given"]

keywords = [
  "mysql with recursive",
  "mysql recursive cte",
  "mysql common table expression",
  "sql recursion",
  "employee hierarchy mysql",
  "recursive queries mysql"
]

description = "A practical look at MySQL CTEs, recursive queries, and the mental model behind WITH RECURSIVE."

showFullContent = false
readingTime = true
hideComments = false
+++

I have worked with SQL for a long time, since I was 18. That is almost 18 years of experience in SQL.

Now when I say SQL, I mean that I started my career doing PL/SQL programming for Oracle databases. Doing data engineering before data engineering was cool lol. I have seen it all over the years: the rise of common table expressions (CTE), massive query planner optimizations, and even a ridiculous 1,000 line UNION style SQL report.

This might seem like I am bragging here and maybe I am a little bit, but it turns out you can always teach an old dog new tricks. As much as I know about Oracle and MySQL databases, I continue to be humbled by this technology every year.

## Why CTEs matter

To understand what I am about to show you, I have to talk about Common Table Expressions. CTE for short (no, not the NFL one). The hell is that?

A CTE (Common Table Expression) is a temporary named result set you define with a WITH clause to make complex SQL queries easier to read and reuse. In MySQL, CTEs became available in version 8.0 (first introduced in 2017 development releases; generally available in 2018).

As you can see it is relatively new. Let me show you why it made writing complex queries easier.

## Old school SQL vs CTEs

Let's take a random example - 

Show me all customers who have spent more than $1,000 in total, along with their names.

Old school style first, you have to use subqueries.

```sql
SELECT c.name, customer_totals.total_spent
FROM (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
) AS customer_totals
JOIN customers c ON customer_totals.customer_id = c.id
WHERE customer_totals.total_spent > 1000;
```

using CTE: 

```sql
WITH customer_totals AS (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
)
SELECT c.name, ct.total_spent
FROM customer_totals ct
JOIN customers c ON ct.customer_id = c.id
WHERE ct.total_spent > 1000;
```

Just looks cleaner this way and easier to read for most people. You might ask that well that first one is not too bad, but let me show you how complicated these queries get. 

Show me all customers who placed orders in the last 90 days, calculate how much each customer spent and how many orders they placed, identify customers who spent more than $5,000 and made at least 5 orders, then return their name, email, total spending, and order count sorted by highest spending.

```sql
SELECT 
  c.name,
  c.email,
  vip_customers.total_spent,
  vip_customers.order_count
FROM (
  SELECT 
    customer_totals.customer_id,
    customer_totals.total_spent,
    customer_order_counts.order_count
  FROM (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    WHERE order_date >= CURDATE() - INTERVAL 90 DAY
    GROUP BY customer_id
  ) AS customer_totals
  JOIN (
    SELECT customer_id, COUNT(*) AS order_count
    FROM orders
    WHERE order_date >= CURDATE() - INTERVAL 90 DAY
    GROUP BY customer_id
  ) AS customer_order_counts
    ON customer_totals.customer_id = customer_order_counts.customer_id
  WHERE customer_totals.total_spent > 5000
    AND customer_order_counts.order_count >= 5
) AS vip_customers
JOIN customers c
  ON vip_customers.customer_id = c.id
ORDER BY vip_customers.total_spent DESC;
```

wtf is this??? I know SQL pretty well and I don't even know what it is doing haha. Imagine trying to come up with that query....
Ok now more like imagine if AI came up with this query.

Let's clean it up with CTEs: 

```sql
WITH recent_orders AS (
  SELECT customer_id, amount, order_date
  FROM orders
  WHERE order_date >= CURDATE() - INTERVAL 90 DAY
),
customer_totals AS (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM recent_orders
  GROUP BY customer_id
),
customer_order_counts AS (
  SELECT customer_id, COUNT(*) AS order_count
  FROM recent_orders
  GROUP BY customer_id
),
vip_customers AS (
  SELECT 
    ct.customer_id,
    ct.total_spent,
    coc.order_count
  FROM customer_totals ct
  JOIN customer_order_counts coc 
    ON ct.customer_id = coc.customer_id
  WHERE ct.total_spent > 5000
    AND coc.order_count >= 5
)
SELECT 
  c.name,
  c.email,
  vc.total_spent,
  vc.order_count
FROM vip_customers vc
JOIN customers c 
  ON vc.customer_id = c.id
ORDER BY vc.total_spent DESC;
```

That select clause looks hella better now. It makes more logical sense. I can picture in my mind that each `AS` block returns a temp memory table and then you join it together.

## Enter `WITH RECURSIVE`

Now let me tell you about a CTE I just learned about that makes no sense in my mind.

```sql
WITH RECURSIVE employee_hierarchy AS (
  SELECT id, name, manager_id, 1 AS level
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  SELECT e.id, e.name, e.manager_id, eh.level + 1
  FROM employees e
  JOIN employee_hierarchy eh
    ON e.manager_id = eh.id
)
SELECT id, name, manager_id, level
FROM employee_hierarchy
ORDER BY level, id;
```

This is my journey into madness with the One SQL Function to Rule them all.

---
## What is recursion? .... What is recursion?

Recursion is a word that I have heard on and off throughout my time in tech. Sounds cool as hell. But every time I heard the word it was followed by, "but never ever use it".

What?! What is wrong with recursion. Apparently a whole lotta things.

The basic idea: a function (or query) calls itself to solve smaller versions of the same problem.

That sounds dope as hell, I want to use it everywhere! Let's solve all the problems a little at a time. Turns out no one wants to risk infinite recursion and absolutely destroying their program.

Recursion is powerful for things like:

walking tree structures
traversing hierarchies
generating sequences
graph/path problems

Really the only one that stands out for normal applications is traversing hierarchies. I think most people would decide that designing the database table better would solve this problem. That could be the case, but there is a famous example of when to use it. This example was in every database class I took. Employee hierarchies.

```sql
CREATE TABLE employees (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  manager_id INT NULL,
  department VARCHAR(100),
  hire_date DATE,
  FOREIGN KEY (manager_id) REFERENCES employees(id)
);
```

As you can see the `manager_id` is a foreign key that references the `id` on its own table. I am sure there are many other complex cases but this is the easier one to show.

## Solving it in code first

How do you generate a hierarchy of employees and who is whose boss. Typically you can write a function, lets use python.

```python
def get_manager_chain(employee_id, employees):
    chain = []

    def trace_manager(emp_id):
        employee = employees.get(emp_id)
        if not employee:
            return

        manager_id = employee.get("manager_id")
        if not manager_id:
            return

        manager = employees.get(manager_id)
        if not manager:
            return

        chain.append(manager["name"])
        trace_manager(manager_id)

    trace_manager(employee_id)
    return chain
```

Looks a little crazy but it makes sense. Keep calling itself until there are no more employees. Just one misplaced return statement and this thing is going to spin to death and crash your process.

Looking at this I pined for the day I could just do this in SQL. For some reason I did not Google it until a coworker told me about a crazy solution to this problem.

## The recursive SQL version

```sql
WITH RECURSIVE manager_chain AS (
  -- Start with the employee
  SELECT 
    id,
    name,
    manager_id,
    1 AS level
  FROM employees
  WHERE id = 5

  UNION ALL

  -- Keep finding each manager up the chain
  SELECT 
    e.id,
    e.name,
    e.manager_id,
    mc.level + 1
  FROM employees e
  JOIN manager_chain mc
    ON e.id = mc.manager_id
)
SELECT id, name, manager_id, level
FROM manager_chain;
```

Mind blown! I stared at it for 10 mins. I thought my super SQL brain would understand this. It in fact did not. How can there be a join in the second part of the union that then joins back to the original CTE. What is going on here?! It took me a long time to understand this mental model.

{{< figure src="know_nothing_anand_siva.png" style="max-width:70%;" >}}

## One important caveat

If there is bad data in your hierarchy, recursive queries can loop forever in spirit even if the database eventually cuts them off. A bad `manager_id` chain can point back to an earlier row and create a cycle.

That is why a depth check is a good idea. Even if you think your data is clean, putting a hard stop on recursion gives you a safety rail when someone loads junk data later.

Here is the same query with a depth check added:

```sql
WITH RECURSIVE manager_chain AS (
  SELECT 
    id,
    name,
    manager_id,
    1 AS level
  FROM employees
  WHERE id = 5

  UNION ALL

  SELECT 
    e.id,
    e.name,
    e.manager_id,
    mc.level + 1
  FROM employees e
  JOIN manager_chain mc
    ON e.id = mc.manager_id
  WHERE mc.level < 10
)
SELECT id, name, manager_id, level
FROM manager_chain;
```

That `WHERE mc.level < 10` line is the guard rail. It means "stop walking the chain after 10 levels no matter what the data says."

## How MySQL is actually evaluating this

Let's see if I can explain this. 

The first thing this statement does is run the anchor query:

```sql
SELECT id, name, manager_id, 1 AS level
FROM employees
WHERE id = 5
```

This just returns employee 5.

So the recursive temp result starts as:

employee 5, manager_id = 4, level 1

Then the SQL statement runs against the current base result.

```sql
SELECT e.id, e.name, e.manager_id, mc.level + 1
FROM employees e
JOIN manager_chain mc
  ON e.id = mc.manager_id
```

Now the accumulated result becomes:

employee 5, manager_id = 4, level 1
employee 4, manager_id = 3, level 2

Recursive part runs again.

Recursive part runs again.

Now MySQL uses the newly found row, employee 4, whose manager_id = 3.

So it finds employee 3.

Now the result becomes:

employee 5, manager_id = 4, level 1
employee 4, manager_id = 3, level 2
employee 3, manager_id = 2, level 3

Then again:

employee 2, manager_id = 1, level 4

Then again:

employee 1, manager_id = NULL, level 5

Then it stops because there is no employee with id = NULL.

I will admit I had to get some ChatGPT to help me map this out, because even now after I wrote this all out, it still looks like voodoo.

Because that is how recursive CTE evaluation works under the hood.

The syntax does look like it might keep joining against the entire accumulated set every time, but the engine evaluates it in rounds:

Run the anchor query once.
Store those rows.
Run the recursive member using the rows found in the previous round.
Add the new rows to the overall result.
Repeat until no new rows are produced.

So there are really two logical buckets:

final accumulated result
current working set from the last iteration

The SQL does not show those buckets explicitly. That part is implied by the rules of recursive CTEs.

So next time you are dealing with a problem and you are looking to write a crazy function to deal with recursion, reach into your bag and pull out WITH RECURSIVE. 

You will surely impress all your database friends (all 3 of them) WITH RECURSIVE 

{{< figure src="matrix.gif" style="max-width:70%;" >}}

{{< figure src="aquarium.gif" style="max-width:70%;" >}}

{{< figure src="dance_party.gif" caption="Me and my database friends" style="max-width:70%;" >}}
