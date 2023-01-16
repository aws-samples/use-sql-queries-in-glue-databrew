## Use SQL queries to define Amazon Redshift datasets in AWS Glue DataBrew

## Solution overview
The following diagram illustrates the architecture for our solution.

![Architecture](/image/BDB-2077-image001.png)

## TABLE1 - create temp customer table

```
CREATE TABLE "public"."customers1"(id character varying(256) encode lzo4,
                                first_name character varying(256) encode lzo,
                                last_name  character varying(256) encode lzo);
```
## Load temp customer table

```
copy "public"."customers1"
from 's3://<S3 path>/customer_primary.csv'
iam_role '<Redshift IAM Role ARN>'
CSV; 

```

## Create customer table to get unique ID 

```
create table customer as 
with asn as (
select * , row_number() over (partition by id order by id asc) as row_number
from customers1 )
select id, first_name, last_name from asn where row_number=1

```

## TABLE2 - create orders table

```
CREATE TABLE "public"."order"(order_id    integer,
                               product_id   integer,
                               customer_id integer,
                               product_name   character varying(256) ,
                               amount      integer,
                               currency   character varying(256) ,
                               order_timestamp  character varying(256),
                               ship_date   character varying(256));

```

## Load orders table

```
copy "public"."order"
from 's3://<S3 path>/order_data.csv'
iam_role '<Redshift IAM Role ARN>'
CSV; 

```

## Custom SQL that considers only USD data

```
JOIN QUERY USED IN CONNECTOR
select cust.first_name, cust.last_name, ord.order_id, ord.product_id, ord.customer_id,  ord.product_name, ord.amount, ord.currency, ord.order_timestamp, ord.ship_date
from order ord, customer cust
where ord.customerid=cust.id(+) and ord.currency='USD'

```

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

