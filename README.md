# mongoDB-learning

c:\Program Files\MongoDB\Server\3.6\bin\

python 3.6, python pip, httpie
#pip install httpie
//check if httpie is installed
#http http://www.google.com


----------
1. Understanding MongoDB
- why MongoDB?
- Document-oriented data (document = JSON)
- Embed or reference?
- Performance
	Sharding - partitions data onto different machines
		Autosharding supported by MongoDB
		Challenging to setup.

2. Explore the system
- Explore the MongoDB shell
	>>mongod   //start mongo deamon

	>>mongo
	>db
	>use learning_mongo  //creates database
	>show dbs
	
	Mongo keeps data as collections of documents
	
	>db.cars.insert({"make":"Subaru"})
	>show collections
	
	Mongo doesn't require schema.
	
	Mongo uses javascript interpeter / javascript expressions.
	
	>print("test")
	>var arr = ["1", "2", "3"]              //you can set variable
	>arr

	You can programmatically interact with database.
	
	>for(i=0;i<1000;i++){
		db.numbers.insert({"number":i})
	}
	
	>db.numbers.count()  //javaScript function
	
	>db.numbers.find({"number":1})
	>db.numbers.find({"number":1}).explain()
	>db.numbers.find({"number":1}).explain("executionStats")
	
	>db.numbers.createIndex({number:1})  //number is a special variable here, it is not a string
	
- Import the data into the database
	(from cmd)
	mongoimport --help
	mongoimport --db learning_mongo --collection tours --jsonArray --file tours.json  
	
	>use learning_mongo
	>show collections
	>db.tours.count()
	>db.tours.find({"tourTags":"hiking"})
	
- Mongo shell operations
	>db.tours.find({"tourTags":"wine"})
	>db.tours.insert({
	"tourName":"The wine of Santa Cruz",
	"tourLength":3,
	"tourDescription":"Discover Santa Cruz's wineries",
	"tourPrice":500,
	"tourTags":["wine", "Santa Cruz"]
	})
	
	//adding region to a tour
	>db.tours.update({"tourName":"The wine of Santa Cruz"},
	{$set:{"tourRegion":"Central Coast"}}
	)
	
	//add a tour tag (add something to an array)
	>db.tours.update({"tourName":"The wine of Santa Cruz"},
	{$addToSet:{"tourTags":"boardwalk"}}
	)
	
	//delete 
	>db.tours.remove({"tourName":"The wine of Santa Cruz"})
	
	//delete collection
	>db.tours.drop()
	
- Simple indexing
	cd 02_04\Start\	
	#mongoimport --db learning_mongo --collection tours --jsonArray --file tours.json
	>mongo
	>use learning_mongo
	>db.tours.find({"tourPackage":"Taste of California"}).explain("executionStats")   --->
			we see that all 29 documents were examined. We don't want that.
	>db.tours.createIndex({tourPackage:1})
	
	//multi-key indexes
	>db.tours.find(
		{
		"tourPrice":{$lte:500},
		 "tourLength":{$lte:3}
		 }
		)
	
	//create compound index
	>db.tours.createIndex({tourPrice:1, tourLength:1})
	//you can create up to 64 indexes per collection in mongo
	
3. Building an Application in Node
-  Node MongoDB setup
	cd 03_01/finished
	>node index.js

	
- Add APIs for read requests in Hapi
	cd 03_02/finished
	>node index.js

	Hapi (server) is listening on port 8080
	>>http http://localhost:8080/api/tours

	In the browser:
	http://localhost:8080/api/tours?tourPackage=Backpack Cal
	
- Add APIs for write requests in Hapi

	>> http POST http://localhost:8080/api/tours tourName="Radek's tour" tourPackage="Fun fun" tourPrice=1000 tourLength=5

	>> http POST "http://localhost:8080/api/tours/Radek's tour" tourBlurb="Get on" replace==true

	>> http DELETE "http://localhost:8080/api/tours/Radek's tour" 

4. Advanced topics
- Unique Indexes
	//remove tours collection
	>db.tours.remove({})
	
	>do.tours.createIndex({"tourName":1,{unique:true}})
	
	//import 2/tours.json
	>db.tours.drop() //drop collection and indexes
	
	
- Tune Mongo queries
	>db.tours.find({}, {tourName:1,_id:0})
	>db.tours.find({}, {tourName:1,tourPrice:1,_id:0}).pretty()
	
	>db.tours.find({}, {tourName:1,tourPrice:1,_id:0}).pretty().sort({tourPrice:-1})//sort descending
	>db.tours.find({}, {tourName:1,tourPrice:1,_id:0}).pretty().sort({tourPrice:-1}).limit(1)
	
	//paging
	>db.tours.find({}, {tourName:1,tourPrice:1,_id:0}).pretty().sort({tourPrice:-1}).limit(1).skip(20)
	
	>db.tours.find({tourPrice:{$lte:1000,$gte:800}}).pretty()
	
- Text indexes
	>db.tours.find({$text:{$search:"wine"}}, {score:$meta:"textScore"}}).pretty().sort({score:$meta:"textScore"})
    >db.tours.find({$text:{$search:"wine"}}, {score:$meta:"textScore", tourName:1,_id:0}}).pretty().sort({score:$meta:"textScore"})
	
	>db.tours.find({tourDescription:{$regex:/backpack/}})
	>db.tours.find({tourDescription:{$regex:/backpack/i}},{tourName:1,_id:0})   //    /i - ignore the case
	>db.tours.find({tourDescription:/backpack/i},{tourName:1,_id:0})
	
- Model your schema
	>var moviename = "Avengers"
	>var movieobj=db.movies.findOne({_id:moviename})
	>movieobj.cast = []
	>var personArray = db.people.find({movies:moviename})
	//iterate over array
	>personArray.forEach(function(person)){
		movieobj.cast.push(person)
	}
	>movieobj
	
- Aggregation
	>db.tours.aggregate([{group:{_id:'$tourPackage', count: {$sum: 1}}}])

	>db.tours.aggregate([{group:{_id:'$tourOrganizer.organizerName', count: {$sum: 1}}}])
	
- Replication and sharding
	Replication - full copies of database
	sharding - partition data onto multiple servers 

	>db.collection.stats()
	>db.stats()
