#synth-datasets

Basically, I just use tdunning's log-synth ( https://github.com/tdunning/log-synth ) to generate some dummy data. Here are some schemas that I've used, along w/ CLI's to generate and parse.


##datasets

###for-rahul
Here are some datasets I made for my buddy rahul.

####orders.schema

this will make ~ XGB of JSON data (which will wind up being ~5GB of CSV). Note the -threads and -output options.

	java -cp ../log-synth/target/log-synth-0.1-SNAPSHOT-jar-with-dependencies.jar com.mapr.synth.Synth -output ../drill/threads -threads 50 -count 70M -schema rahul/schemas/orders.json -format JSON

this will produce output similar to:

	{"order_id":1090,"month":"January","date":"2014-01-30 06:09:04","cust_id":3614,"prod_id":1,"order_total":82,"z":{"zip":"34656","longitude":"-81.7443","latitude":"28.8686"}}
	{"order_id":1091,"month":"January","date":"2014-01-08 22:16:37","cust_id":1572,"prod_id":421,"order_total":7,"z":{"zip":"77520","longitude":"-94.1442","latitude":"30.6082"}}
	{"order_id":1092,"month":"January","date":"2014-01-15 09:02:34","cust_id":3063,"prod_id":660,"order_total":26,"z":{"zip":"27690","longitude":"-78.6372","latitude":"36.0165"}}
	{"order_id":1093,"month":"January","date":"2014-01-28 20:37:48","cust_id":
	
Note that it will make 50 files of ~ 240MBytes each.  To make life easier for drill:

	for i in `ls`;do mv $i $i.json;done;


	

Now, lets have drill turn the output into CSV (since the CSV output option from log-synth will still leave JSON in the mix because of the zip code information):

sql:

	create or replace view ord_view as select t.order_id as order_id,t.`month` as `month`, t.`date` as ord_date, t.cust_id as cust_id, t.prod_id as prod_id, t.order_total as order_total, t.z.zip as zipcode, t.z.longitude as longitude, t.z.latitude as latitude from `threads` t
	
	
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
	
that's one big hurk'in CSV file, probably makes sense to split it up, how about 1mm lines per file? (70 files)

	split -l 1000000 test.csv drill/threads/csv/orders.segment
	
then this again:

	for i in `ls`;do mv $i $i.csv;done;
	
Perhaps we should take the quotes away?:

	for i in `ls`;do sed -r -i 's/\x27//g' $i;done;
	
	
then make a view in sqlline:
	
	create or replace view ord_csv_view as select columns[0] as order_id, columns[1] as `month`, columns[2] as ord_date, columns[3] as cust_id, columns[4] as prod_id, columns[5] as ord_total, columns[6] as zip, columns[7] as longitude, columns[8] as latitude from `threads/csv`;
	
	

