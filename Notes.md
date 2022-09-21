# MongoDB Notes - Day 1
​
## Databases:
- SQL vs NoSQL 
- SQL (MySQL, Postgres, Sqlite):
	- All the databases that use the SQL query language
	- Invented in the 70's, mainly toward business users and developers
	- Aggregate and study data
	- Organized by rows and columns
	- Weakness: 
		- SQL needs a table schema defined before you can use the database. I.E. it is hard to update a table with a new field after creating it
​
- Users Table
	- Name, Address, Email, Phone
	- James, 100 Something Ave, james@gmail.com, 1235897189
	- Ginny, 200 Something ave, ginny@gmail.com, 7858623468
​
- NoSQL (MongoDB, Cassandra):
	- All the databases that do not use the SQL query language
- MongoDB
	- Organized by Document reference
	- Strength:
		- Mongo can accept new fields very easily and update all documents in the table with the new field. I.E. It is easy to make changes to your data after creating it
- Users Table
	- {
		name: James,
		address: 100 something ave,
		email: james@gmail.com,
		phone: 123574791,
		zip: 01829
	}
​
- TL;DR
	- The web development world tends to work in MongoDB because it is very easy to change the shape of the data after you create the database. 
​
## MongoDB:
- Organization
	- Cluster
		- A cluster is the computer/server/data center that your database is running on
		- When we connect to MongoDB, we are making a real time 2 way SSH connection to the cluster in order to communicate with our database
		- A cluster can have multiple databases
	- Database
		- The database is the storage mechanism of the cluster. I.E. The database is where we store our data for a specific project.
	- Collection
		- The collection is the list of specific pieces of data that you are storing. E.G. For a BlogDB, we would store users, blogs, categories.
	- Document
		- The document is the individual piece of data that we are storing, modifing and retriving. E.G. a blog post
		- Technically documents are stored as BSON, but you can just think of it as JSON
- _Note_: We will be using Mongo Atlas which is the cloud based version of MongoDB
​
- When it comes to connecting to your Mongo cluster, generally we only want to whitelist IP addresses that we know are secure, this is a security measure. 
	- For us, it is easier to simply allow access from anywhere while we're learning MongoDB. But in the future, it is good practice to only whitelist your local IP address as well as any deployed application IP addresses.
- When we connect to our cluster, we will still need to provide a username + password which will act as a layer of security for us.
- Usually during development, we use GUI applications to connect to our database so that we can run queries and see our data directly. E.G. NoSQLBooster
- Connection string example (note <password> must be replaced with your password):
- mongodb+srv://jnissenbaum:<password>@codeimmersivescluster.d6fvp.mongodb.net/?retryWrites=true&w=majority
- Basic .find() query
	- db.users.find({})
  .projection({})
  .sort({_id:-1})
  .limit(100)
	- db is a global variable in our query playground (NoSQLBooster) that is a reference to our database (E.G. Users or Blogs)
	- users is the collection within our database that we are running the query against
	- .find({}) is the method we are invoking for the query
	- Then we can chain other methods onto .find({}) such as:
		- .projection({}) which will select specific fields to display
		- .sort({}) which will sort the result by some criteria
		- .limit(n) which will limit our result to n number of documents
- The .find({}) method takes a single object as its first argument, which can be used to query specific parts of our dataset. E.G. db.users..find({
	firstName: "James"
}) will match all documents in our collection which has a firstName field that has the value of "James"
- _Note_: If we pass in an empty object {} into .find() it will return us the entire dataset
- Example query, find all documents whose lastModified date is greater than "2022-09-17T17:27:19.392Z"
	- db.users.find({
			lastModified: {
					$gt: "2022-09-17T17:27:19.392Z"
			}
		})
- Mongo Docs Cheat Sheet
	- https://www.mongodb.com/developer/products/mongodb/cheat-sheet/
​
# MongoDB Notes - Day 2
​
- MongoDB has native types such as string, boolean, date, double(number), array, etc
- All functions in NoSQLBooster are just generated query scripts. Meaning that all functionality can be written and exectued by a user without needing NoSQLBooster
- What is the _id field?
	- Every single document in a collection must be unique and the _id field is how Mongo ensures that every document has a unique identifier
	- The _id field is an internal MongoDB field that is automatically generated for each document in the collection and it is used by Mongo to ensure that all documents are unique even if all the other fields have been duplicated in a different entry
	- The _id field MUST be a Mongo ObjectId type
	- _Recommendation_: Do not work with the _id field if you can because it ends up being more trouble than it's worth 
- What is a uuid?
	- A uuid is a unique identifier generated by us (the developers) that looks something like this: "edde9278-97d9-4bc1-9cf1-aa67c6269b38"
	- _Critically_: A uuid generated with a uuid library is guaranteed to be unique and it allows to identify a specific entry in our collection
- Mongo stores documents as BSON in which each field can be given a specific type as opposed to JSON in which all fields must be strings
- .find() Query
	- The object that is passed into the .find() method can take Mongo operators for specific operations on a field. E.G. if we were looking for the number field age equal to 31 we would do:
		- db.users.find({
			age: {
				$eq: 31
			}
		})
		- We nest an object with the $operator for the specific field we are looking for
		- Basic Comparison Operators
		- $eq
			- equals
		- $ne
			- not-equals
		- $gt/e
			- greater than ($gt), greater than or equal to ($gte)
			- _Note_: For dates greater than will be any date AFTER the date provided
		- $lt/e
			- less than ($lt), less than or equal to ($lte)
			- _Note_: For dates less than will be any date BEFORE the date provided
		- $in
			- look into an array of values and search for documents with matching values in the specified field
		- $nin
			- Opposite of $in, searched for entries whose specified array field does NOT have the given array values
		- $exists
			- $exists takes boolean as the value to check all documents in which the given field either exists for true or does not exist for false
			- This operator comes in handy when you have different versions of data in your collection. E.G. If I was working with a version of user data that had firstName + lastName and then I switched over to using only fullName, then could search for all the old versions of the users in order to update them.
		- $type
			- $type checks for any documents in which the given field matches the specified type
			- _Note_: $exists and $type are useful for finding old versions of the data objects so that we can delete them or update them to the new version
		- $regex
			- $regex will do a regular expression match on a string
			- To construct a simple regular expression, we put the string we want to search for in between two forward slashes
				- {
					$regex: /@gmail.com/
				}
- _Side-Note_: UNIX Timestamps
	- Behind the scenes, every date object is just a UNIX Timestamp
	- A UNIX Timestamp is calculated by the amount of milliseconds that have occurred after January 01, 1970
	- For every date after the current date will be a larger/greater UNIX timestamp and every date before the current date will be a smaller/lower UNIX timestamp

    # MongoDB Notes - Day 3

## Updating Documents
- The two primary methods for updating documents in the database is .updateMany() for multiple document updates and .updateOne() for a single document update
- The .update methods take two object arguments, the query object and the update object
	- The query object is the first argument and is where you write the query for the documents you want to find and update
	- The update object is the second argument and defines the operations you want mongo to execute to update the found document/documents. The update object has several update operators.
- Field Update Operators
	- $set
		- $set will overwrite the specified field with the new value given. The value for $set will be an object, the keys will be the fields on the document you want to update and the values will be the new values for those fields. E.G. To set the field firstName, the update object would be:
			- {
				$set: {
					firstName: "James"
				}
			}
		- _Note_: $set will set the value for a field regardless of whether or not the field existed on the object before the update. I.E. we can use $set to update either an existing field or make a new field on the document.
- Array Update Operators
	- $push/$pull
		- $push and $pull work similarly to how the JS .push() and .pop()(like .pop but for an item in any position in the array) methods work where you can push a new value onto an array of values or you can remove a value from the array. E.G. To push a new movie onto the favoriteMovies array you would do:
			- {
				$push: {
					favoriteMovies: "Alien"
				}
			}
		- _Note_: $pull has the same syntax as $push
	- $addToSet
		- $addToSet works like $push except it will NOT push a duplicate value into the array
	- $each
		- For pushing in an array of items into an array, we use $each to loop through the given array of values and push each one in turn
			- {
				$addToSet: {
					favoriteMovies: {
						$each: ["Sunshine", "Fight Club"]
					}
				}
			}
## Deleting Documents
- The two primary methods for deleting objects are .deleteOne() and .deleteMany()
	- These two methods only take a single argument which will be the query object. Once executed, the delete methods will find any documents matching the query and delete them.

- _Note_: All methods that specifically match a single document such as updateOne and deleteOne will match the first document found and run the operation. If there are multiple matching documents and a sort criteria is not defined, mongo will find the first one at "random"