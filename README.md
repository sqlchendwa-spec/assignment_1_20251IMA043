# Assignment 1: Database Management & SQL Analytics
**Student Name:** Kundwa Nkwaya Kelche  
**Student ID:** 20251IMA043  
**Database Tool:** PostgreSQL (via pgAdmin 4)

---

## 1. Business Scenario Summary
Sunrise Supermarket is a local grocery store chain operating in Rwanda, with customers across cities like Kigali, Musanze, Huye, and Rubavu. This analysis uses SQL to help store management answer basic business questions: which products sell best, how often repeat customers come back to buy groceries, who our top big-spenders are, and how overall sales are growing over time.

---

## 2. Database Schema & Setup

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price NUMERIC(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date DATE
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT
);
###3. SQL Queries, Explanations & Results
**##Query 1: INNER JOIN (Orders + Customers)
**Explanation: Connects orders to customers so we can see who bought what and which city they live in.

SELECT o.order_id, c.customer_name, c.city, o.order_date
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
<img width="527" height="500" alt="sunrise_mrkt_q1" src="https://github.com/user-attachments/assets/58464c9b-7321-47ba-bdb7-c8418e85d475" />

**Query 2: JOIN (Order Items + Products)
**Explanation: Matches items inside an order with the product table to show the product name, category, price, and quantity bought.

SELECT oi.order_item_id, oi.order_id, p.product_name, p.category, p.price, oi.quantity
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id;

<img width="724" height="620" alt="sunrise_mrkt_q2" src="https://github.com/user-attachments/assets/b2400a9a-4451-49e2-9533-a34a672f71b4" />

**Query 3: LEFT JOIN (Customers + Orders)
**Explanation: Lists all customers, including those who have signed up but haven't placed an order yet (like Evan Wright).

SELECT c.customer_id, c.customer_name, o.order_id, o.order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

<img width="467" height="450" alt="sunrise_mrkt_q3" src="https://github.com/user-attachments/assets/1af37d7a-4cf2-4f20-a3b5-fc253f132f16" />

**Query 4: CTE + Filtering (Above Average Spend)
**Explanation: Calculates how much money each customer spent in total, and then filters to show only the big spenders who spent more than the average customer.

WITH CustomerSpend AS (
    SELECT c.customer_id, c.customer_name, SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT *
FROM CustomerSpend
WHERE total_spent > (SELECT AVG(total_spent) FROM CustomerSpend);

<img width="418" height="90" alt="sunrise_mrkt_q4" src="https://github.com/user-attachments/assets/8e20b33d-21a3-4114-b302-64c8fbbe8aab" />


**Query 5: Ranking Customers by Spend (Window Function)
**Explanation: Ranks customers from 1st place downwards based on who spent the most money at the store.
WITH CustomerSpend AS (
    SELECT c.customer_id, c.customer_name, SUM(oi.quantity * p.price) AS total_spent
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT customer_id, customer_name, total_spent,
       RANK() OVER (ORDER BY total_spent DESC) AS spend_rank
FROM CustomerSpend;

<img width="513" height="162" alt="sunrise_mrkt_q5" src="https://github.com/user-attachments/assets/b8f91270-ac6d-4dc1-a250-27062d0c3731" />


**Query 6: Numbering Orders Chronologically (Window Function)
**Explanation: Numbers each customer's orders in order (1st order, 2nd order, 3rd order) to see their shopping history over time.

SELECT customer_id, order_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_number
FROM orders;

<img width="454" height="424" alt="sunrise_mrkt_q6" src="https://github.com/user-attachments/assets/5465433b-f210-4c98-b733-faeeb6a1f383" />

**Query 7: Running Revenue Over Time (Window Function)
**Explanation: Calculates daily store sales and keeps a running total to show how total income accumulates day after day.

WITH DailyRevenue AS (
    SELECT o.order_date, SUM(oi.quantity * p.price) AS daily_total
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY o.order_date
)
SELECT order_date, daily_total,
       SUM(daily_total) OVER (ORDER BY order_date) AS running_total_revenue
FROM DailyRevenue;

<img width="362" height="371" alt="sunrise_mrkt_q7" src="https://github.com/user-attachments/assets/98fa72af-0fb3-4437-b262-c4c7a238f2c4" />

**Query 8: Days Between Customer Orders (Window Function)
**Explanation: Calculates how many days passed between a customer's current order and their previous order to see how frequently they return.

SELECT customer_id, order_id, order_date,
       order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_last_order
FROM orders;

<img width="482" height="410" alt="sunrise_mrkt_q8" src="https://github.com/user-attachments/assets/153fbbb2-0877-4834-8cf5-b4064f9740e6" />

###4. Business Insights & Interpretation
Customer Spending: A few repeat shoppers drive most of the sales. On the other hand, signed-up customers who haven't bought anything (eg: Evan Wright) show that the store needs special discounts or welcome messages to get them to make their first purchase.

Shopping Frequency: Regular shoppers come back every 2 - 5 days, showing that people rely on the supermarket for frequent  needs like milk, bread, and fresh produce.

Overall Revenue: Total sales grew steadily throughout September without long dry spells, showing a healthy daily stream of store revenue.

##5. Technical Challenges & Resolutions
pgAdmin Disconnections: The software timed out and disconnected when left sitting idle. This was fixed simply by clicking the connection button to refresh the database session.

Understanding Window Functions: Writing functions like RANK() and LAG() was confusing at first, but using a WITH clause (CTE) helped break the steps down into simple, easy-to-read parts.

