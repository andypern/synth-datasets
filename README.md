#synth-datasets

Basically, I just use tdunning's log-synth ( https://github.com/tdunning/log-synth ) to generate some dummy data. Here are some schemas that I've used, along w/ CLI's to generate and parse.


##datasets

###for-rahul
Here are some datasets I made for my buddy rahul.

####orders.schema

	java -cp log-synth/target/log-synth-0.1-SNAPSHOT-jar-with-dependencies.jar com.mapr.synth.Synth -count 100 -schema orders.schema -format CSV

this will produce output similar to:

	1090,"January","2014-01-13 22:20:42",3989,1,35,{"zip":"34656","state":"FL","longitude":"-81.7443","latitude":"28.8686"}
	1091,"January","2014-01-26 00:29:29",2946,68,15,{"zip":"77520","state":"TX","longitude":"-94.1442","latitude":"30.6082"}
	1092,"January","2014-01-19 02:50:31",1432,63,43,{"zip":"27690","state":"NC","longitude":"-78.6372","latitude":"36.0165"}
	1093,"January","2014-01-30 19:48:17",2169,21,73,{"zip":"41845","state":"KY","longitude":"-81.9598","latitude":"37.1439"}


This is nice, but for a CSV..not so much.  Options:

1.  Write a script to post-process
2.  make the output JSON instead of CSV, and use drill to generate the output :)

sql:

	create or replace view ord_view as select t.order_id as order_id,t.`month` as `month`, t.`date` as ord_date, t.cust_id as cust_id, t.prod_id as prod_id, t.order_total as order_total, t.z.zip as zipcode, t.z.state as state, t.z.longitude as longitude, t.z.latitude as latitude from `log.json` t
	

