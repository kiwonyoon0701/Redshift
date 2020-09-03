## Redshift Data Load & Unload

## SCT Agent

## DML

## Copy from s3
Create table data.seoul_public_bicycle_rental_place_info(
	idx BIGINT,
	district VARCHAR(30),
	rental_place_id VARCHAR(30),
	rental_place_num VARCHAR(30),
	rental_place_name VARCHAR(300),
	bicycle_cnt BIGINT,
	latitude NUMERIC(10,6),
	longitude NUMERIC(10,6)
	) DISTSTYLE KEY
	DISTKEY (RENTAL_PLACE_NUM)
	SORTKEY (IDX);

copy data.seoul_public_bicycle_rental_place_info
FROM 's3://s3-learn-redshift-kiwony/seoul_public_bicycle_rental_place_info/'
CREDENTIALS 'aws_access_key_id=;aws_secret_access_key='
FORMAT
DELIMITER ','
NULL AS ''
IGNOREHEADER 1
REMOVEQUOTES;


Create table data.seoul_public_bicycle_usage(
	date VARCHAR(10),
	hour VARCHAR(2),
	rental_place_num VARCHAR(30),
	rental_place_name VARCHAR(300),
	rental_category_code VARCHAR(20),
	gender VARCHAR(2),
	age_code VARCHAR(6),
	usage_count INT,
	momentum NUMERIC(10,2),
	carbon_emmision NUMERIC(10,2),
	travel_distance_meter BIGINT,
	travel_time_min BIGINT
)DISTSTYLE KEY
DISTKEY(rental_place_num)
SORTKEY(date);

copy data.seoul_public_bicycle_usage
FROM 's3://s3-learn-redshift-kiwony/seoul_public_bicycle_usage/'
CREDENTIALS 'aws_access_key_id=;aws_secret_access_key='
FORMAT
DELIMITER ','
NULL AS ''
IGNOREHEADER 1
REMOVEQUOTES;




## EMR

## DynamoDB

## SSH endpoint

## Kinesis Firehose

## Unload

## Unload using Glue
