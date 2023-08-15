# Case Study #6 Clique Bait

*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-6/).*
![Screenshot 2023-08-13 at 8 38 37 pm](https://github.com/jef-fortunahamid/CaseStudy6_CliqueBait/assets/125134025/18fbe525-22c6-401f-a97f-82adb9abae03)

## Business Task
Clique Bait" is an unconventional online seafood store, founded by Danny. With a background in digital data analytics, Danny aims to merge his expertise with the seafood business.
Our objective is to assist Danny in realizing his vision for Clique Bait. We need to delve into the dataset provided, analyze it, and devise innovative methods to determine the funnel fallout rates for the online store.

## General Insights
- *Impression Effectiveness:* The analysis revealed the influence of ad impressions on user behavior. By comparing visits with and without impressions, we can determine if impressions lead to increased engagement, cart additions, or purchases.
- *Click-to-Purchase Conversion:* Focusing on whether clicking an ad leads to a purchase, this insight provides a clear metric on the effectiveness of ad designs and calls to action.
- *Uplift in Purchase Rate:* This sheds light on the nuances of impressions. It considers not only whether an impression leads to a purchase, but also contrasts it with other user groups, giving a holistic view of ad campaign efficacy.
- *Campaign Success Metrics:* By comparing different campaigns, this section allows the marketing team to identify which campaigns were the most successful and why, based on metrics like page views, cart adds, and purchases.
- *User Journey Analysis:* A deep dive into user behavior, this section examines the sequence of product additions to the cart, helping the team understand user preferences and possibly predicting future behavior.

## Key SQL Syuntax and Functions

## Questions and Solutions
### Part A. Enterprise Relationship Diagram

> Using the following DDL schema details to create an ERD for all the Clique Bait datasets.
> [Click here](https://dbdiagram.io/) to access the DB Diagram tool to create the ERD.

```sql
CREATE TABLE clique_bait.event_identifier (
  "event_type" INTEGER,
  "event_name" VARCHAR(13)
);

CREATE TABLE clique_bait.campaign_identifier (
  "campaign_id" INTEGER,
  "products" VARCHAR(3),
  "campaign_name" VARCHAR(33),
  "start_date" TIMESTAMP,
  "end_date" TIMESTAMP
);

CREATE TABLE clique_bait.page_hierarchy (
  "page_id" INTEGER,
  "page_name" VARCHAR(14),
  "product_category" VARCHAR(9),
  "product_id" INTEGER
);

CREATE TABLE clique_bait.users (
  "user_id" INTEGER,
  "cookie_id" VARCHAR(6),
  "start_date" TIMESTAMP
);

CREATE TABLE clique_bait.events (
  "visit_id" VARCHAR(6),
  "cookie_id" VARCHAR(6),
  "page_id" INTEGER,
  "event_type" INTEGER,
  "sequence_number" INTEGER,
  "event_time" TIMESTAMP
);
```
*Solution*
```sql
TABLE clique_bait.event_identifier {
  event_type integer
  event_name varchar
}

TABLE clique_bait.campaign_identifier {
  campaign_id integer
  products varchar
  campaign_name varchar
  start_date timestamp
  end_date timestamp
}

TABLE clique_bait.page_heirarchy {
  page_id integer
  page_name varchar
  product_category varchar
  product_id integer
}

TABLE clique_bait.users {
  user_id integer
  cookie_id varchar
  start_date timestamp
}

TABLE clique_bait.events {
  visit_id varchar
  cookie_id varchar
  page_id varchar
  event_type integer
  sequence_number integer
  event_time timestamp
}


Ref: "clique_bait"."events"."event_type" > "clique_bait"."event_identifier"."event_type"

Ref: "clique_bait"."events"."cookie_id" - "clique_bait"."users"."cookie_id"

Ref: "clique_bait"."events"."page_id" > "clique_bait"."page_heirarchy"."page_id"

```
![Screenshot 2023-08-05 at 9 45 11 pm](https://github.com/jef-fortunahamid/CaseStudy6_CliqueBait/assets/125134025/8b64a5b2-8d9f-42cb-ad71-4c0fbe7a3321)

### Part B: Digital Analysis
> 1. How many users are there?
```sql
SELECT 
  COUNT(DISTINCT user_id) AS unique_user_count
FROM clique_bait.users;
```

> 2. How many cookies does each user have on average?
```sql
WITH user_cookie_count AS (
  SELECT 
      user_id
    , COUNT(cookie_id) AS cookie_count_per_user
  FROM clique_bait.users
  GROUP BY user_id
)
SELECT
  ROUND(AVG(cookie_count_per_user), 2) AS avg_cookie_count
FROM user_cookie_count;
```

> 3. What is the unique number of visits by all users per month?
```sql
SELECT
    DATE_TRUNC('month', event_time)::DATE AS month_start
  , COUNT(DISTINCT visit_id) AS unique_visits
FROM clique_bait.events
GROUP BY month_start
ORDER BY month_start;
```

> 4. What is the number of events for each event type?
```sql
SELECT
    events.event_type
  , event_identifier.event_name
  , SUM(1) AS count_events
FROM clique_bait.events
INNER JOIN clique_bait.event_identifier
  ON events.event_type = event_identifier.event_type
GROUP BY
    events.event_type
  , event_identifier.event_name
ORDER BY events.event_type;
```

> 5. What is the percentage of visits which have a purchase event?
```sql
WITH visits_with_purchase AS (
  SELECT
      visit_id
    , SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase_flag
  FROM clique_bait.events
  GROUP BY visit_id
)
SELECT
  ROUND(100 * SUM(purchase_flag) / COUNT(*), 2) AS purchase_percentage
FROM visits_with_purchase;
```

> 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
WITH checkout_purchase_visit AS (
  SELECT
      visit_id
    , MAX(CASE WHEN event_type = 1 and page_id = 12 THEN 1 ELSE 0 END) AS checkout_flag
    , MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase_flag
  FROM clique_bait.events
  GROUP BY visit_id
)
SELECT
  ROUND(100 * SUM(CASE WHEN purchase_flag = 0 THEN 1 ELSE 0 END)::NUMERIC/ COUNT(*), 2) AS checkout_without_purchase_percentage
FROM checkout_purchase_visit
WHERE checkout_flag = 1
```

> 7. What are the top 3 pages by number of views?
```sql
SELECT
    t2.page_name
  , COUNT(1) AS page_views
FROM clique_bait.events AS t1 
INNER JOIN clique_bait.page_hierarchy as t2 
  ON t1.page_id = t2. page_id
WHERE event_type = 1
GROUP BY t2.page_name
ORDER BY page_views DESC
LIMIT 3;
```

> 8. What is the number of views and cart adds for each product category?
```sql
SELECT
    page_hierarchy.product_category
  , SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS page_views
  , SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM clique_bait.events
INNER JOIN clique_bait.page_hierarchy
  ON events.page_id = page_hierarchy.page_id
WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY page_views DESC;
```

> 9. What are the top 3 products by purchases?
```sql
WITH purchase_visits AS (
  SELECT
    visit_id
  FROM clique_bait.events
  WHERE event_type = 3
)
SELECT
    page_hierarchy.product_id
  , page_hierarchy.page_name AS product_name
  , SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS purchases
FROM clique_bait.events 
INNER JOIN clique_bait.page_hierarchy 
  ON events.page_id = page_hierarchy.page_id
WHERE EXISTS (
  SELECT NULL
  FROM purchase_visits
  WHERE events.visit_id = purchase_visits.visit_id
)
AND page_hierarchy.product_id IS NOT NULL
GROUP BY 
    page_hierarchy.product_id
  , page_hierarchy.page_name
ORDER BY page_hierarchy.product_id;
```

### Part C: Product Funnel Analysis

>  Using a single SQL query - create a new output table which has the following details:
> - How many times was each product viewed?
> - How many times was each product added to cart?
> - How many times was each product added to a cart but not purchased (abandoned)?
> - How many times was each product purchased?
> 
> Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

**Table 1**
```sql
DROP TABLE IF EXISTS product_info;
CREATE TEMP TABLE product_info AS
WITH product_page_events AS (
  SELECT
      events.visit_id
    , page_hierarchy.product_id
    , page_hierarchy.page_name
    , page_hierarchy.product_category
    , SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_view
    , SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_add
  FROM clique_bait.events
  INNER JOIN clique_bait.page_hierarchy
    ON events.page_id = page_hierarchy.page_id
  WHERE page_hierarchy.product_id IS NOT NULL
  GROUP BY
      events.visit_id
    , page_hierarchy.product_id
    , page_hierarchy.page_name
    , page_hierarchy.product_category
),
visit_purchase AS (
  SELECT DISTINCT
    visit_id
  FROM clique_bait.events
  WHERE event_type = 3
),
combined_product_events AS (
  SELECT
      t1.visit_id
    , t1.product_id
    , t1.page_name
    , t1.product_category
    , t1.page_view
    , t1.cart_add
    , CASE WHEN t2.visit_id IS NOT NULL THEN 1 ELSE 0 END AS purchase
  FROM product_page_events AS t1
  LEFT JOIN visit_purchase  AS t2
    ON t1.visit_id = t2.visit_id
)
SELECT
    product_id
  , page_name AS product 
  , product_category
  , SUM(page_view) AS page_views
  , SUM(cart_add) AS cart_adds
  , SUM(CASE WHEN cart_add = 1 AND purchase = 0 THEN 1 ELSE 0 END) AS abandoned
  , SUM(CASE WHEN cart_add = 1 AND purchase = 1 THEN 1 ELSE 0 END) AS purchases
FROM combined_product_events
GROUP BY product_id, product, product_category 
ORDER BY product_id;

SELECT * FROM product_info;
```

**Table 2**
```sql
DROP TABLE IF EXISTS product_category_info;
CREATE TEMP TABLE product_category_info AS 
SELECT
    product_category
  , SUM(page_views) AS page_views
  , SUM(cart_adds) AS cart_adds
  , SUM(abandoned) AS abandoned
  , SUM(purchases) AS purchases
FROM product_info
GROUP BY product_category;

SELECT * FROM product_category_info;
```

> 1. Which product had the most views, cart adds and purchases?
```sql
SELECT 
    product 
  , page_views
FROM product_info
ORDER BY page_views DESC
LIMIT 1;

SELECT 
    product 
  , cart_adds
FROM product_info
ORDER BY cart_adds DESC
LIMIT 1;

SELECT 
    product 
  , purchases
FROM product_info
ORDER BY purchases DESC
LIMIT 1;
```

> 2. Which product was most likely to be abandoned?
```sql
SELECT
    product 
  , ROUND(abandoned::NUMERIC / cart_adds, 2) AS abandoned_likelihood
FROM product_info
ORDER BY abandoned_likelihood DESC
LIMIT 1;
```

> 3. Which product had the highest view to purchase percentage?
```sql
SELECT
    product 
  , ROUND(100 * purchases / page_views, 2) AS view_to_purchase_percentage
FROM product_info
ORDER BY percentage DESC
LIMIT 1;
```
> 4. What is the average conversion rate from view to cart add?
```sql
SELECT
  ROUND(AVG(100 * cart_adds / page_views), 2) AS avg_view_to_cart_add
FROM product_info;
```

> 5. What is the average conversion rate from cart add to purchase?
```sql
SELECT
  ROUND(AVG(100 * purchases / cart_adds), 2) AS avg_cart_add_to_purchase
FROM product_info;
```

### Part D: Campaign Analysis
> Generate a table that has 1 single row for every unique visit_id record and has the following columns:
> - `user_id`
> - `visit_id`
> - `visit_start_time`: the earliest `event_time` for each visit
> - `page_views`: count of page views for each visit
> - `cart_adds`: count of product cart add events for each visit
> - `purchase`: 1/0 flag if a purchase event exists for each visit
> - `campaign_name`: map the visit to a campaign if the `visit_start_time` falls between the `start_date` and `end_date`
> - `impression`: count of ad impressions for each visit
> - `click`: count of ad clicks for each visit
> - **(Optional column)** `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the `sequence_number`)
> 
> Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.
> 
> Some ideas you might want to investigate further include:
> - Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression > event
> - Does clicking on an impression lead to higher purchase rates?
> - What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
> - What metrics can you use to quantify the success or failure of each campaign compared to eachother?

```sql
DROP TABLE IF EXISTS visit_summary;
CREATE TEMP TABLE visit_summary AS
SELECT
    users.user_id
  , events.visit_id
  , MIN(events.event_time) AS visit_start_time
  , SUM(CASE WHEN events.event_type = 1 THEN 1 ELSE 0 END) AS page_views
  , SUM(CASE WHEN events.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
  , MAX(CASE WHEN events.event_type = 3 THEN 1 ELSE 0 END) AS purchase
  , campaign_identifier.campaign_name
  , SUM(CASE WHEN events.event_type = 4 THEN 1 ELSE 0 END) AS impression
  , SUM(CASE WHEN events.event_type = 5 THEN 1 ELSE 0 END) AS click
  , STRING_AGG(
          CASE WHEN page_hierarchy.product_id IS NOT NULL AND event_type = 2
              THEN page_hierarchy.page_name
            ELSE NULL END,
          ', ' ORDER BY events.sequence_number
        ) AS cart_products
  FROM clique_bait.events
  INNER JOIN clique_bait.users
    ON events.cookie_id = users.cookie_id
  LEFT JOIN clique_bait.campaign_identifier
    ON events.event_time 
          BETWEEN campaign_identifier.start_date 
          AND campaign_identifier.end_date
  LEFT JOIN clique_bait.page_hierarchy
    ON events.page_id = page_hierarchy.page_id
  GROUP BY
      users.user_id
  , events.visit_id
  , campaign_identifier.campaign_name;

SELECT * FROM visit_summary LIMIT 10;
```
> 1. Impression Effectiveness:
> - Compare the average page views, cart adds, and purchases for visits with impressions (impression > 0) versus visits without impressions (impression = 0).
```sql
WITH with_without_impression AS (
  SELECT
      'with_impression' AS impression
    , ROUND(AVG(page_views),2) AS avg_page_views
    , ROUND(AVG(cart_adds), 2) AS avg_cart_adds
    , ROUND(AVG(purchase), 2) AS avg_purchase
  FROM visit_summary
  WHERE impression > 0
UNION
  SELECT
      'without_impression' AS impression
    , ROUND(AVG(page_views),2) AS avg_page_views
    , ROUND(AVG(cart_adds), 2) AS avg_cart_adds
    , ROUND(AVG(purchase), 2) AS avg_purchase
  FROM visit_summary
  WHERE impression = 0
)
SELECT *
FROM with_without_impression;
```
