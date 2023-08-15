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




