Transaction Search Demo
========================

This demo traces moving vehicles as they pass through geohash tiles. It also keeps track of a vehicle movements on a day to day basis. Similar to a vessel tracking or taxi application.  

The application 

1. Allows the user to track a vehicles movements per day.

2. Find all vehicles per tile. Tiles have 2 sizes. Tile1 is large, Tile2 is small. 

3. Find all vehicles within a given radius of any vehicle

To specify contact points use the contactPoints command line parameter e.g. '-DcontactPoints=192.168.25.100,192.168.25.101'
The contact points can take mulitple points in the IP,IP,IP (no spaces).
 
To create the schema, run the following

	mvn clean compile exec:java -Dexec.mainClass="com.datastax.demo.SchemaSetup" -DcontactPoints=localhost
	
To create the solr core, run 

	dsetool create_core datastax_taxi_app.current_location reindex=true coreOptions=src/main/resources/solr/rt.yaml schema=src/main/resources/solr/geo.xml solrconfig=src/main/resources/solr/solrconfig.xml
	
To continuously update the locations of the vehicles run 
	
	mvn clean compile exec:java -Dexec.mainClass="com.datastax.taxi.Main" -DcontactPoints=localhost
	
To start the web server, in another terminal run 

	mvn jetty:run
	
To find all movements of a vehicle use http://localhost:8080/datastax-taxi-app/rest/getmovements/{vehicle}/{date} e.g.

	http://localhost:8080/datastax-taxi-app/rest/getmovements/1/20160112

Or

	select * from vehicle where vehicle = '1' and day='20160112';

To find all vehicle movement, use the rest command http://localhost:8080/datastax-taxi-app/rest/getvehicles/{tile} e.g.

	http://localhost:8080/datastax-taxi-app/rest/getvehicles/gcrf

or 

	CQL - select * from current_location where solr_query = '{"q": "tile1:gcrf"}' limit 1000;


To find all vehicles within a certain distance of a latitude and longitude, http://localhost:8080/datastax-taxi-app/rest/search/{lat}/{long}/{distance}

	http://localhost:8080/datastax-taxi-app/rest/search/52.53956077140064/-0.20225833920426117/5
	
Or

	select * from current_location where solr_query = '{"q": "*:*", "fq": "{!geofilt sfield=lat_long pt=52.53956077140064,-0.20225833920426117 d=5}"}' limit 1000;
 	
To remove the tables and the schema, run the following.

    mvn clean compile exec:java -Dexec.mainClass="com.datastax.demo.SchemaTeardown"
    
    
