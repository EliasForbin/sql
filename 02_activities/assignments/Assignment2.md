# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `April 27, 2025`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack at `#cohort-6-help`. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

***

## Section 1:
You can start this section following *session 1*, but you may want to wait until you feel comfortable wtih basic SQL query writing. 

Steps to complete this part of the assignment:
- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3


###  Design a Logical Model

#### Prompt 1
Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized. 

Additionally, include a date table. 

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a conceptual model only. 

#### Prompt 2
We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.

#### Prompt 3
The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2? 

**HINT:** search type 1 vs type 2 slowly changing dimensions. 

```
We can attempt two physical design examples with the proposed architectures. 

Type 1: Overwiting changes to the CUSTOMER_ADDRESS table. This is simpler and requires fewer resources, making it useful where only the current address is needed. CustomerID here can act as the primary key.
SQL Table Schema
CREATE TABLE Customer_Address (
    CustomerID INT PRIMARY KEY,         -- Unique identifier for the customer
    AddressLine1 VARCHAR(255) NOT NULL,
    AddressLine2 VARCHAR(255),
    City VARCHAR(100) NOT NULL,
    State VARCHAR(100) NOT NULL,
    PostalCode VARCHAR(20) NOT NULL,
    Country VARCHAR(100) NOT NULL
);


Type 2: Retaining changes to the CUSTOMER_ADDRESS table. This porvides flexibility to track and query historical addresses but inlvolves more complexity and storage. Here it will be important to have an AddressID as the primary key. Additionally, StartDate and EndDate attributes track the time period during with the address was valid. IsCurrent can act as a binary attribute to add an additional layer to identify the most recent address for quick querying puposes.
SQL Table Schema
CREATE TABLE Customer_Address (
    AddressID INT PRIMARY KEY AUTO_INCREMENT, -- Unique identifier for each address entry
    CustomerID INT NOT NULL,                  -- Links to the customer
    AddressLine1 VARCHAR(255) NOT NULL,
    AddressLine2 VARCHAR(255),
    City VARCHAR(100) NOT NULL,
    State VARCHAR(100) NOT NULL,
    PostalCode VARCHAR(20) NOT NULL,
    Country VARCHAR(100) NOT NULL,
    StartDate DATE NOT NULL,                  -- When the address became active
    EndDate DATE DEFAULT NULL,                -- When the address was replaced (NULL if current)
    IsCurrent BOOLEAN DEFAULT TRUE,           -- Indicates if this is the active address
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID) -- Links to Customer table
);

***

## Section 2:
You can start this section following *session 4*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question


### Write SQL

#### COALESCE
1. Our favourite manager wants a detailed long list of products, but is afraid of tables! We tell them, no problem! We can produce a list with all of the appropriate details. 

Using the following syntax you create our super cool and not at all needy manager a list:
```
SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product
```

But wait! The product table has some bad data (a few NULL values). 
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first problem, and 'unit' for the second problem. 

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

<div align="center">-</div>

-- Create the product table
CREATE TABLE product (
    product_id INTEGER PRIMARY KEY,-- Unique ID for each product in our list
    product_name VARCHAR(255),-- Product name
    product_size VARCHAR(50),-- Size or the weight of the product.
    product_qty_type VARCHAR(50)-- Type of quantity (e.g. bottle, piece, bag etc)
);

-- Insert sample product data
INSERT INTO product (product_name, product_size, product_qty_type) VALUES
('Banana', 'Medium', 'Piece'),
('Milk', '1 Liter', 'Bottle'),
(NULL,NULL, 'Bag'),
('Body Lotion', '500 ml', 'Bottle'),
('Bread', 'Large', 'Loaf'),
('Coffee', '250 g', 'Pack'),
('Tea', '200 g', 'Tin'),
('Mango Juice', '1 Liter', 'Bottle'),
('Pasta', '500 mg', 'package'),
('Cheese', '500 g', 'Block');

-- Retrieve the product list in the desired format
SELECT 
    coalesce(product_name, ' ') || ', ' || coalesce(product_size, 'unit') || ' (' || product_qty_type || ')' AS product_details
FROM product;

#### Windowed Functions
1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. 

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

--Numbering each visit (Unsing Row_Number)
SELECT 
    customer_id,
    market_date,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date) AS visit_number
FROM new_customer_purchases;


--Numbering Unique Market dates
SELECT 
    customer_id,
    market_date,
    DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY market_date) AS visit_number
FROM new_customer_purchases;


2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

--QUESTION 2
--Reverse numbering
SELECT 
    customer_id,
    market_date,
	vendor_id, 
	product_id, 
	quantity,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS reversed_visit_number
FROM new_customer_purchases;

--Use subquery to fliter most recent visit
SELECT customer_id,
FROM (SELECT customer_id, market_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS reversed_visit_number
    FROM new_customer_purchases
)AS visit_data
WHERE visit_data.reversed_visit_number = 1;


3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.

<div align="center">-</div>
SELECT 
    customer_id,
    product_id,
    market_date,
    COUNT(*) OVER (PARTITION BY customer_id, product_id) AS purchase_count
FROM new_customer_purchases;

#### String manipulations
1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. 

<div align="center">-</div>

SELECT
    product_name,
    CASE 
      WHEN INSTR(product_name, '-') > 0 
      THEN TRIM(SUBSTR(product_name, INSTR(product_name, '-') + 1))
      ELSE NULL
    END AS product_description
FROM product;

#### UNION
1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them. 

--Drop the tenporary table if it already EXISTS
DROP TABLE IF EXISTS TempDailySales;
--Create a temp table that aggregates total sales per market date
CREATE TEMP TABLE TempDailySales AS
    SELECT 
        market_date, 
        SUM(quantity*cost_to_customer_per_qty) AS sales
    FROM customer_purchases
    GROUP BY market_date
--SELECT* FROM TempDailySales;
--Create a tenp table with rank windowed function on the previous query to create best day and worst day
DROP TABLE IF EXISTS RankedDays;
CREATE TEMP TABLE RankedDays AS 
    SELECT 
        market_date,
        sales,
        DENSE_RANK() OVER (ORDER BY sales DESC) AS best_rank,
        DENSE_RANK() OVER (ORDER BY sales ASC)  AS worst_rank
    FROM TempDailySales;
--SELECT* FROM RankedDays;
--Query the second table twice, one for best and the other for worst with a union bing them.
SELECT 
    market_date,
    sales,
	'Highest' AS sales_type
	FROM RankedDays
	WHERE best_rank=1
UNION
SELECT 
    market_date,
    sales,
	'Lowest' AS sales_type
	FROM RankedDays
	WHERE worst_rank=1;


***

## Section 3:
You can start this section following *session 5*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question

### Write SQL

#### Cross Join
1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** customer on record. How much money would each vendor make per product? Show this by vendor_name and product name, rather than using the IDs.

**HINT**: Be sure you select only relevant columns and rows. Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. Think a bit about the row counts: how many distinct vendors, product names are there (x)? How many customers are there (y). Before your final group by you should have the product of those two queries (x\*y). 

SELECT 
    v.vendor_name,
    p.product_name,
    5 * vi.original_price * cust.total_customers AS potential_revenue
FROM vendor_inventory AS vi
JOIN vendor AS v 
  ON vi.vendor_id = v.vendor_id
JOIN product AS p
  ON vi.product_id = p.product_id
CROSS JOIN (
    SELECT COUNT(*) AS total_customers FROM customer
) AS cust;

<div align="center">-</div>

#### INSERT
1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  Name the timestamp column `snapshot_timestamp`.

CREATE TABLE product_units AS
SELECT 
    p.*,
    CURRENT_TIMESTAMP AS snapshot_timestamp
FROM product p
WHERE product_qty_type = 'unit';

2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie). 

INSERT INTO product_units (product_id, product_name, product_size, product_category_id, product_qty_type, snapshot_timestamp)
VALUES (24, 'Apple Pie', 'Large',1, 'unit', CURRENT_TIMESTAMP);

<div align="center">-</div>

#### DELETE 
1. Delete the older record for the whatever product you added.

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

DELETE FROM product_units
WHERE product_name = 'Apple Pie'
  AND snapshot_timestamp = (
    SELECT MAX(snapshot_timestamp)
    FROM product_units
    WHERE product_name = 'Apple Pie'
  );

<div align="center">-</div>

#### UPDATE
1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.
```
ALTER TABLE product_units
ADD current_quantity INT;
```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details. 

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.

UPDATE product_units
SET current_quantity = COALESCE((
    SELECT MAX(quantity)
    FROM vendor_inventory
    WHERE vendor_inventory.product_id = product_units.product_id
), 0)
WHERE product_id IN (
    SELECT DISTINCT product_id
    FROM vendor_inventory
);
SELECT* FROM product_units;