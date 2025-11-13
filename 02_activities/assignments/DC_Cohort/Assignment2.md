# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `August 17, 2025`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1 and 4
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

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
Your answer...
```

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
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first column with nulls, and 'unit' for the second column with nulls. 

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

<div align="center">-</div>

SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product
WHERE product_name is NULL 
OR product_size IS NULL 
OR product_qty_type is NULL;

SELECT 
COALESCE(product_name, ' ') || ', ' || product_size || ' (' || COALESCE(product_qty_type, 'unit') || ')'
FROM product;

#### Windowed Functions
1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. 

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

SELECT
	customer_id
	,market_date
,ROW_NUMBER () OVER ( PARTITION BY CUSTOMER_ID ORDER BY MARKET_DATE) AS visit_number
FROM customer_purchases; 

SELECT
	customer_id
	,market_date
,dense_rank () OVER ( PARTITION BY CUSTOMER_ID ORDER BY MARKET_DATE) AS visit_number
FROM customer_purchases; 

2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

SELECT
	customer_id
	,market_date
,ROW_NUMBER () OVER ( PARTITION BY CUSTOMER_ID ORDER BY MARKET_DATE DESC) AS visit_number
FROM customer_purchases;

SELECT 
    customer_id,
    market_date
FROM (
    SELECT
        customer_id,
        market_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY market_date DESC) AS visit_number
    FROM customer_purchases
) AS numbered_visits
WHERE visit_number = 1;

3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.

SELECT DISTINCT
	customer_id, 
	product_id, 
	market_date, 
	COUNT (*) OVER (PARTITION BY customer_id, product_id) AS purchase_counter
FROM customer_purchases;

<div align="center">-</div>

#### String manipulations
1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. 

SELECT 
    product_name,
    CASE 
        WHEN INSTR(product_name, '-')  
        THEN TRIM(SUBSTR(product_name, INSTR(product_name, '-') +1))
        ELSE NULL
    END AS description
FROM product;

2. Filter the query to show any product_size value that contain a number with REGEXP. 

SELECT 
	product_name,
	product_size
From product
WHERE product_size REGEXP '[0-9]';

<div align="center">-</div>

#### UNION
1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them. 

WITH sales_by_date AS (
    SELECT
        md.market_date,
        SUM(cp.quantity * cp.cost_to_customer_per_qty) AS sales
    FROM
        customer_purchases AS cp
        INNER JOIN market_date_info AS md
            ON cp.market_date = md.market_date
    GROUP BY
        md.market_date
),
ranked_sales AS (
    SELECT
        market_date,
        sales,
        RANK() OVER (ORDER BY sales DESC) AS highest_rank,
        RANK() OVER (ORDER BY sales ASC)  AS lowest_rank
    FROM
        sales_by_date
)

SELECT
    market_date,
    sales
FROM
    ranked_sales
WHERE
    highest_rank = 1

UNION

SELECT
    market_date,
    sales
FROM
    ranked_sales
WHERE
    lowest_rank = 1;

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
FROM ( vendor_inventory vi
JOIN vendor v 
    ON vi.vendor_id = v.vendor_id
JOIN product p 
    ON vi.product_id = p.product_id
)
CROSS JOIN  ( (SELECT COUNT(*) FROM customer) * 5 * vi.original_price ) AS total_revenue
ORDER BY v.vendor_name, p.product_name;


<div align="center">-</div>

#### INSERT
1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  Name the timestamp column `snapshot_timestamp`.

DROP TABLE IF EXISTS temp.product_units;
CREATE TEMP TABLE product_units AS
SELECT 
			*,
    CURRENT_TIMESTAMP AS snapshot_timestamp
FROM
    product
WHERE
    product_qty_type = 'unit';


2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie). 

INSERT INTO product_units
VALUES ('3241', 'Apple Pie', '20"', '32', 'unit', CURRENT_TIMESTAMP);

<div align="center">-</div>

#### DELETE 
1. Delete the older record for the whatever product you added.

DELETE FROM product_units
WHERE product_id = 7;
SELECT * 
FROM product_units;

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

<div align="center">-</div>

#### UPDATE
1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.

ALTER TABLE product_units
ADD current_quantity INT;
```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details. 

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.
*** 

ALTER TABLE product_units
ADD current_quantity INT;

UPDATE product_units as pu
SET current_quantity = COALESCE(
    (
        SELECT vi.quantity
        FROM vendor_inventory vi
        WHERE vi.product_id = pu.product_id
        ORDER BY vi.market_date DESC, vi.vendor_id DESC
        LIMIT 1
    ),
    0
);

## Section 4:
You can start this section anytime.

Steps to complete this part of the assignment:
- Read the article
- Write, within this markdown file, <1000 words.

### Ethics

Read: Boykis, V. (2019, October 16). _Neural nets are just people all the way down._ Normcore Tech. <br>
    https://vicki.substack.com/p/neural-nets-are-just-people-all-the

**What are the ethical issues important to this story?**

Consider, for example, concepts of labour, bias, LLM proliferation, moderating content, intersection of technology and society, ect. 


```
There are several ethical issues that are important to this story that the author wants us to consider. The first one is that training neural networks to recognize different objects requires a lot of human labour. These labourers, such as students, can be exploited for their hidden work. Also the work that is used to train the neural networks are unrecognized for their efforts as this could take thousands of hours. The second consideration builds upon the idea that because ML models are trained by humans, there are biases in the type of data used dependening on the socio-economic status, geographical location, and culture. For instance, in North America, we may percieve someone as "rich" if they are in a expensive car. However, in asian countires someone "rich" could be surrounded by a huge loving family. This leads to a lot of bias in the way models are trained. Additionally, people may percieve machines as autonomous but in actuality, this is just an illusion because there is a lot of work behind to scene to achieve this. When a machine has reached a stage of autonomy, it has typically undergone thousands of hours of training. This can instill fear in people that robots are going to automate all jobs. Moving forward, I believe that research development teams behind these automated systems should be more open and transparent about how their models are trained and about the human effort behind them. Companies deploying these models also have a responsibility to communicate this to users or consumers so that society can better understand the labour, potential biases, and limitations embedded in these technologies! 
```
