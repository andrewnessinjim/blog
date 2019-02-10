---
published: false
---
# Hello World with Java in Heroku and MongoDB in Atlas
We'll create a java application with a main method that exposes a web service. This service responds with a message fetched from an M0 instance running in [MongoDB Atlas][1]. You can download the code from here. All the tools used are free.

## Prerequisites
[Maven][4]
[JDK][5]

## Step 1: Build and Run a Jersey Hello World Application
By the end of this step, you should have a java application that you can run locally and has a web service which returns a hardcoded `Hello World` string.

Run the below command in a new empty directory:

    mvn archetype:generate -DarchetypeArtifactId=jersey-heroku-webapp \
                    -DarchetypeGroupId=org.glassfish.jersey.archetypes -DinteractiveMode=false \
                    -DgroupId=com.ripecode -DartifactId=mongo-heroku-webapp -Dpackage=com.ripecode \
                    -DarchetypeVersion=2.28
                
You can change the `groupId`, `artifactId` and `package` values as required. This will create an application with this structure:

    │   pom.xml
    │   Procfile
    │   system.properties
    │
    └───src
        └───main
            ├───java
            │   └───com
            │       └───ripecode
            │           │   MyResource.java
            │           │
            │           └───heroku
            │                   Main.java
            │
            └───webapp
                └───WEB-INF
                        web.xml

I've ommitted the test directory. In fact, remove the test directory, we're not concerned with it in this tutorial. We'll talk about each file in a moment. For now, let's see the application in action. Run the below commands to build and run the application:

    cd mongo-heroku-webapp/
    mvn clean package
    java -cp "target/classes;target/dependency/*" com.ripecode.heroku.Main (For Windows)
    java -cp target/classes:target/dependency/* com.ripecode.heroku.Main (For Linux)
    
You should see a message similar to this:

    2019-02-10 19:54:44.489:INFO:oejs.ServerConnector:main: Started ServerConnector@24105dc5{HTTP/1.1}{0.0.0.0:8080}

You should now be able to access the URL exposed by the application in this URL: [http://localhost:8080/myresource](http://localhost:8080/myresource)

## Step 2: Spin Up Your Mongod Instance on Atlas
Follow the first 5 steps from [MongoDB's official docs][6] to start up your free cluster. It is a 3 member replica set and allows up to 512MB storage. We could do a lot of interesting demo projects with that :).

Now you should have your free instance running and a user with admin role. It's best that we create a separate user for our application. This user usually has read and write access to a particular database, because any application is meant to do just that in most cases. For our example, we'll create such an user:

1. Click the Security tab from the Clusters view from your atlas account.
2. Click on MongoDB roles.
3. Click on "+ Add New Custom Role" button.
4. Fill up the values as shown:
![MongoDB Atlas New Custom Role]({{ site.baseurl }}/images/hello-mongo-heroku/new_role.png)
Leaving the collection field empty means the user has relevant access to all the collections in the mentioned database.
5. Now head back to the MongoDB Users subtab and click on "+ ADD NEW USER".
6. Fill up th values as shown:
![MongoDB Atlas New User]({{ site.baseurl }}/images/hello-mongo-heroku/new_user.png)

You are ready to test your instance now. Head back to the Overview tab in your atlas account and click on the CONNECT button. Follow the steps to connect to your instance from a Mongo shell. Remember to use the username and password of the new user we created. Insert a sample document once you're connected:

    MongoDB Enterprise Cluster0-shard-0:PRIMARY> use tutorial
    switched to db tutorial
    MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.messages.insertOne({message: "Hello from MongoDB"})
    {
            "acknowledged" : true,
            "insertedId" : ObjectId("5c60b282f066fe5f423c76b6")
    }
    MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.messages.findOne()
    {
            "_id" : ObjectId("5c60b282f066fe5f423c76b6"),
            "message" : "Hello from MongoDB"
    }

This is the document we're going to fetch from our simple java application

## Step 3: Connect To Your Atlas mongod Instance From Java
Add the below dependency to your `pom.xml` file:

    <dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongodb-driver</artifactId>
      <version>3.9.1</version>
    </dependency>

1. Head back to your atlas account and click on CONNECT button from the Overview tab.
2. Choose Connect Your Application.
3. Copy the standard connection string.
4. Replace the username and password to tutorial-user. This is the same user created earlier. The URL should look something like this now:


      mongodb://tutorial-user:tutorial-user@cluster0-shard-00-00-2lbue.mongodb.net:27017,cluster0-shard-00-01-2lbue.mongodb.net:27017,cluster0-shard-00-02-2lbue.mongodb.net:27017/test?ssl=true&replicaSet=Cluster0-shard-0&authSource=admin
  
You can refer to the [docs][7] for a step by step tutorial, but here's the code for connecting:

    package com.ripecode.heroku;

    import com.mongodb.MongoClient;
    import com.mongodb.MongoClientURI;
    import com.mongodb.client.MongoCollection;
    import com.mongodb.client.MongoDatabase;
    import org.bson.Document;
    import org.bson.types.ObjectId;

    public class MongoService {
        private static  MongoClient mongoClient;

        static {
            mongoClient = new MongoClient(new MongoClientURI("mongodb://tutorial-user:tutorial-user@cluster0-shard-00-00-2lbue.mongodb.net:27017,cluster0-shard-00-01-2lbue.mongodb.net:27017,cluster0-shard-00-02-2lbue.mongodb.net:27017/test?ssl=true&replicaSet=Cluster0-shard-0&authSource=admin"));
        }

        public static String getMessage() {
            MongoDatabase database = mongoClient.getDatabase("tutorial");
            MongoCollection<Document> collection = database.getCollection("messages");


            Document query = new Document("_id", new ObjectId("5c60b282f066fe5f423c76b6"));
            Document result = collection.find(query).iterator().next();


            return result.getString("message");
        }
    }

Place this code in a new `MongoService.java` file in `com.ripecoe.heroku` package. Change your `getIt()` method in `MyResource` class:

    @GET
        @Produces(MediaType.TEXT_PLAIN)
        public String getIt() {
            return MongoService.getMessage();
        }
    
 Add the necessary import:
 
 	import com.ripecode.heroku.MongoService;
    
 You can now run the same commands you used before and see the application in action:
 
    cd mongo-heroku-webapp/
    mvn clean package
    java -cp "target/classes;target/dependency/*" com.ripecode.heroku.Main (For Windows)
    java -cp target/classes:target/dependency/* com.ripecode.heroku.Main (For Linux)
    
  You should now be able to access the URL exposed by the application in this URL: [http://localhost:8080/myresource](http://localhost:8080/myresource). But this time you should see the message you inserted in MongoDB.

## References
* [Heroku's official "Getting Started on Heroku with Java" tutorial][2]
* [Jersey's official "Creating a Web Application that can be deployed on Heroku" tutorial][3]

[1]: https://www.mongodb.com/cloud/atlas
[2]: https://devcenter.heroku.com/articles/getting-started-with-java
[3]: https://jersey.github.io/documentation/latest/getting-started.html#heroku-webapp
[4]: https://maven.apache.org/install.html
[5]: https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/
[6]: https://docs.atlas.mongodb.com/getting-started/
[7]: http://mongodb.github.io/mongo-java-driver/3.4/driver/getting-started/quick-start/