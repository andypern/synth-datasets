#synth-datasets

Basically, I just use tdunning's log-synth ( https://github.com/tdunning/log-synth ) to generate some dummy data. Here are some schemas that I've used, along w/ CLI's to generate and parse.


##datasets

###for-rahul
Here are some datasets I made for my buddy rahul.

####orders.schema

this will make ~ XGB of JSON data (which will wind up being ~5GB of CSV): (perhaps it makes sense to use the threads option..)

	java -cp ../log-synth/target/log-synth-0.1-SNAPSHOT-jar-with-dependencies.jar com.mapr.synth.Synth -count 61M -schema rahul/schemas/orders.json -format JSON > ../drill/orders.json

this will produce output similar to:

	{"order_id":1090,"month":"January","date":"2014-01-30 06:09:04","cust_id":3614,"prod_id":1,"order_total":82,"z":{"zip":"34656","longitude":"-81.7443","latitude":"28.8686"}}
	{"order_id":1091,"month":"January","date":"2014-01-08 22:16:37","cust_id":1572,"prod_id":421,"order_total":7,"z":{"zip":"77520","longitude":"-94.1442","latitude":"30.6082"}}
	{"order_id":1092,"month":"January","date":"2014-01-15 09:02:34","cust_id":3063,"prod_id":660,"order_total":26,"z":{"zip":"27690","longitude":"-78.6372","latitude":"36.0165"}}
	{"order_id":1093,"month":"January","date":"2014-01-28 20:37:48","cust_id":
	

Now, lets have drill turn the output into CSV (since the CSV output option from log-synth will still leave JSON in the mix because of the zip code information):

sql:

	create or replace view ord_view as select t.order_id as order_id,t.`month` as `month`, t.`date` as ord_date, t.cust_id as cust_id, t.prod_id as prod_id, t.order_total as order_total, t.z.zip as zipcode, t.z.longitude as longitude, t.z.latitude as latitude from `orders.json` t
	
	
query it to make sure it looks right:
	
	select * from ord_view limit 3;
	

should show:

	+------------+------------+------------+------------+------------+-------------+------------+------------+------------+------------+
	|  order_id  |   month    |  ord_date  |  cust_id   |  prod_id   | order_total |  zipcode   |   state    | longitude  |  latitude  |
	+------------+------------+------------+------------+------------+-------------+------------+------------+------------+------------+
	| 1000       | January    | 2014-01-02 07:23:48 | 2313       | 66         | 26          | 72132      | null       | -92.1202   | 34.6595    |
	| 1001       | January    | 2014-01-04 01:59:03 | 3836       | 0          | 64          | 17307      | null       | -76.3022   | 39.9701    |
	| 1002       | January    | 2014-01-15 23:22:05 | 2101       | 79         | 54          | 66216      | null       | -94.6277   | 39.0715    |
	+------------+------------+------------+------------+------------+-------------+------------+------------+------------+------------+
	

Exit sqlline and make a file called 'runme.sql' containing:


	select * from ord_view;
	
now run sqlline:

	sqlline --run=runme.sql --outputformat=csv > test.csv
	

