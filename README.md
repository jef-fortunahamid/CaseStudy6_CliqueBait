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
> # Part A. Enterprise Relationship Diagram

Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

[Click here](https://dbdiagram.io/) to access the DB Diagram tool to create the ERD.
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
```
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


















