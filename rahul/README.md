#for-rahul
Here are some datasets I made for my buddy rahul.

##orders

this will make ~ XGB of JSON data (which will wind up being ~500MB of CSV). Note the -threads and -output options.

	java -cp log-synth/target/log-synth-0.1-SNAPSHOT-jar-with-dependencies.jar \
	 com.mapr.synth.Synth \
	 -count 7M \
	 -threads 50 \
	 -schema synth-datasets/rahul/schemas/orders.1week.json \
	 -format JSON \
	 -output drill/rahul/orders/json

this will produce output similar to:

	{"order_id":1000,"month":"January","date":"2014-01-04 23:39:48","cust_id":99,"prod_id":11,"order_total":18,"z-zip":"96206","z-longitude":"","z-latitude":""}
	{"order_id":1001,"month":"January","date":"2014-01-01 00:34:41","cust_id":620,"prod_id":39,"order_total":15,"z-zip":"76530","z-longitude":"-97.44","z-latitude":"30.71"}
	{"order_id":1002,"month":"January","date":"2014-01-05 17:33:46","cust_id":162,"prod_id":48,"order_total":29,"z-zip":"63460","z-longitude":"-92.20","z-latitude":"40.01"}
	{"order_id":1003,"month":"January","date":"2014-01-03 04:57:51","cust_id":789,"prod_id":198,"order_total":60,"z-zip":"83809","z-longitude":"-116.59","z-latitude":"48.06"}
	
Note that it will make 50 files of ~ 23MBytes each.  To make life easier for drill:
	
	cd drill/rahul/orders/json
	for i in `ls`;do mv $i $i.json;done;


	
Note: this data is in JSON format, we want it in CSV (soon enough Ted's tool will properly output this into CSV, but not quite yet..).  In order to get it into CSV, lets use Drill:

	sqlline
	
switch workspaces:

	use maprfs.apernsteiner;
	
query it to make sure it looks right:
	
	select * from `rahul/orders/json` limit 3;
	

should show:


	+------------+------------+------------+------------+------------+-------------+------------+-------------+------------+
	|  order_id  |   month    |    date    |  cust_id   |  prod_id   | order_total |   z-zip    | z-longitude | z-latitude |
	+------------+------------+------------+------------+------------+-------------+------------+-------------+------------+
	| 1014       | January    | 2014-01-07 20:52:08 | 5725       | 400        | 39          | 46186      | -85.60      | 39.88      |
	| 1037       | January    | 2014-01-05 17:34:28 | 25         | 87         | 62          | 66061      | -94.81      | 38.88      |
	| 1067       | January    | 2014-01-01 07:40:27 | 122        | 8          | 35          | 04490      | -67.73      | 45.41      |
	+------------+------------+------------+------------+------------+-------------+------------+-------------+------------+
	


In order to get CTAS to output to CSV, you need to change the output format (for your session) to cdv:

	alter session set `store.format` = 'csv';	

	
Run the following to create a directory and table within..

	create table maprfs.apernsteiner.`products_CSV` as
	select order_id, `date`, cust_id, prod_id, order_total, `z-zip` as zip,
	`z-longitude` as longitude, `z-latitude` as latitude
	from maprfs.apernsteiner.`/rahul/orders/json` order by `date`;
	
		
Exit sqlline, and check: that there is a file:
	
	ls -lh products_CSV/0_0_0.csv
	-rwxr-xr-x 1 mapr mapr 378M Apr 15 22:18 BEST/0_0_0.csv
	
That's fine for now, perhaps for larger sets (1 year or more) we can split up by date.  Note that if you don't do an 'order by' in your CTAS statement, it will auto-fragment according to some internal record limit in drill.



To synth more (1 year worth), use `orders.year.json` , which has a start/end of 01/01/2014 => 12/31/2014 .	 If you want to keep the volume consistent, just set the row count to 365M when running log-synth.  To 'escalate' the # of orders as the year progresses, you would have to synth one month or quarter at a time.  More on that later.


 
	
##customers

Here's a snip of customers.csv , which is synth'd via:

	java -cp ../log-synth/target/log-synth-0.1-SNAPSHOT-jar-with-dependencies.jar \
	com.mapr.synth.Synth \
	-count 45000 \
	-schema rahul/schemas/customers.json \
	-format CSV > ../drill/customers.csv


output:
	
	42799,"James Berger","48342","OTHER","21-25","501-1000","415-72-7667","gold"
	9967,"Arthur Richards","79061","FEMALE","26-35","501-1000","467-44-7261","silver"
	364,"Ron Watson","38721","FEMALE","26-35","3001-100000","003-90-5003","silver"

column headers:

	cust_id,name,zip,gender,age_range,spend_range,son,member_status
	
Note: I left the double quotes ("") in, perhaps that will cause issues w/ upstream tools..but that's part of the fun :)

	
Note that we have created 45,000 cust_ids (45,000 rows of output), randomly distributed.  There are 50,000 cust_id's in the orders table, also randomly distributed (with some skew), so there will be 5,000 random cust_id's in the orders table that don't have a match (this is purposeful)


##Products

Products I cheated..I didn't use log-synth, I grabbed some 'real world' product data (about 949 entries) from an office supply table.  Note that I added a few items to make it more interesting.  Here's sample output:

	963,"HP Pavilion Touchsmart 11z",laptop,369
	964,"MSI GT70",laptop,1399
	965,"Razer Blade",laptop,2399
	
column headers:
	
	prod_id,prod_name,prod_cat,price

