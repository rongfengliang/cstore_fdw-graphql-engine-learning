# pg_cron &&cstore && hasura graphql-engine learning


## pg_cron

```code
CREATE EXTENSION pg_cron;
CREATE TABLE apps (
    id SERIAL PRIMARY KEY,
    insert_date timestamp without time zone
);
SELECT cron.schedule('* * * * *','insert into apps(insert_date) values(now())');
SELECT * FROM cron.job;
SELECT cron.unschedule(cronid) FROM cron.job;
```

## cstore

```code
wget http://examples.citusdata.com/customer_reviews_1998.csv.gz
wget http://examples.citusdata.com/customer_reviews_1999.csv.gz

gzip -d customer_reviews_1998.csv.gz
gzip -d customer_reviews_1999.csv.gz

-- load extension first time after install
CREATE EXTENSION cstore_fdw;

-- create server object
CREATE SERVER cstore_server FOREIGN DATA WRAPPER cstore_fdw;

-- create foreign table
CREATE FOREIGN TABLE customer_reviews
(
    customer_id TEXT,
    review_date DATE,
    review_rating INTEGER,
    review_votes INTEGER,
    review_helpful_votes INTEGER,
    product_id CHAR(10),
    product_title TEXT,
    product_sales_rank BIGINT,
    product_group TEXT,
    product_category TEXT,
    product_subcategory TEXT,
    similar_product_ids CHAR(10)[]
)
SERVER cstore_server
OPTIONS(compression 'pglz');

\COPY customer_reviews FROM 'customer_reviews_1998.csv' WITH CSV;
\COPY customer_reviews FROM 'customer_reviews_1999.csv' WITH CSV;

SELECT
    customer_id, review_date, review_rating, product_id, product_title
FROM
    customer_reviews
WHERE
    customer_id ='A27T7HVDXA3K2A' AND
    product_title LIKE '%Dune%' AND
    review_date >= '1998-01-01' AND
    review_date <= '1998-12-31';

```

## hasura graphql-engine

* add track for customer_reviews

* some query

```code
query {
  customer_reviews (where:{
    customer_id:{
      _eq:"A27T7HVDXA3K2A"
    },
    product_title:{
      _like:"%Dune%"
    },
    review_date:{
      _gte:"1998-01-01",
      _lte:"1998-12-31"
    }
  }) {
    customer_id
    review_date
    product_id
    review_rating
    product_title
  }
}
```