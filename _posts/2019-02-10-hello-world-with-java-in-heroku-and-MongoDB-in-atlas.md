---
published: true
title: Hello World with Java in Heroku and MongoDB in Atlas
---
We'll create a java application with a main method that exposes a web service using [jersey library][12]. This service responds with a message fetched from an M0 instance running in [MongoDB Atlas][1]. You can download the code from [here][13]. All the tools used are free.

## Prerequisites
* [Install Maven][4]
* [Install JDK][5]
* [Install Git][8]

## Step 1: Build and Run a Hello World Java Application using Jersey
By the end of this step, you should have a java application that you can run locally and has a web service which returns a hardcoded `Hello World` string.

Run the below command in a new empty directory on your computer:

<script src="https://gist.github.com/andrewnessinjim/fb8636ec3fae63c5944f8b241febd1fc.js"></script>
                
You can change the `groupId`, `artifactId` and `package` values as required. This will create an application with this structure:

<script src="https://gist.github.com/andrewnessinjim/0cbeca1dbc5125fda1c88c9901a7c1cd.js"></script>

I've omitted the test directory. In fact, remove the test directory, we're not concerned with it in this tutorial. We'll talk about some of these files in a moment. For now, let's see the application in action. Run the below commands to build and run the application:

<script src="https://gist.github.com/andrewnessinjim/23f94d0f5e4cb11284a50397e20aa3e8.js"></script>
    
You should see a message similar to this:

<script src="https://gist.github.com/andrewnessinjim/7a95295b8c83af19560c5197d93d12d4.js"></script>

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
6. Fill up the values as shown:

![MongoDB Atlas New User]({{ site.baseurl }}/images/hello-mongo-heroku/new_user.png)

You are ready to test your instance now. Head back to the Overview tab in your atlas account and click on the CONNECT button. Follow the steps to connect to your instance from a Mongo shell. Remember to use the username and password of the new user we created. Insert a sample document once you're connected:

<script src="https://gist.github.com/andrewnessinjim/f431e101c16f7d8fc95eff4bb5d1ccbc.js"></script>

This is the document we're going to fetch from our simple java application

## Step 3: Connect To Your Atlas mongod Instance From Java
Add the below dependency to your `pom.xml` file:

<script src="https://gist.github.com/andrewnessinjim/e60fd31b3b5c91b6c0399b7da315b646.js"></script>

1. Head back to your atlas account and click on CONNECT button from the Overview tab.
2. Choose Connect Your Application.
3. Copy the standard connection string.
4. Replace the username and password to tutorial-user. This is the same user created earlier. The URL should look something like this now:

<script src="https://gist.github.com/andrewnessinjim/402bd69f25f11e76a354b0cfbd4be437.js"></script>
    
You can refer to the [docs][7] for a step by step tutorial, but here's the code for connecting:

<script src="https://gist.github.com/andrewnessinjim/3f762c431f38f6852b5383afc33218e7.js"></script>

Place this code in a new `MongoService.java` file in `com.ripecoe.heroku` package. Change your `getIt()` method in `MyResource` class:

<script src="https://gist.github.com/andrewnessinjim/1595b5ad5ca880dc018982f37ab9fb96.js"></script>
    
 Add the necessary import:
 
`import com.ripecode.heroku.MongoService;`
    
 You can now run the same commands you used before and see the application in action:
 
<script src="https://gist.github.com/andrewnessinjim/23f94d0f5e4cb11284a50397e20aa3e8.js"></script>
    
  You should now be able to access the URL exposed by the application in this URL: [http://localhost:8080/myresource](http://localhost:8080/myresource). But this time you should see the message you inserted in MongoDB.

## Step 4: Prepare The Application for Heroku
Now let's get back to the files  we saw earlier:

<script src="https://gist.github.com/andrewnessinjim/8065cb53443f780a9f129db49d703cd9.js"></script>
    
The `ProcFile` tells Heroku how to run your application. This file will have the same java command that we have been using to run the application so far. The `pom.xml` file is configured to dump all your application dependencies into a directory called `dependency` under `target`. Heroku makes the `target` directory available when the commands in `ProcFile` are executed. Hence, we're able to mention subdirectories of `target` folder as java classpaths. You need not change any of this. However, ensure that the JDK version configured in your `system.properties` file is `1.8`:

`java.runtime.version=1.8`

This is essential to connect to MongoDB free tier instance. Heroku deployments are carried out using git repositories. So we've to create a git repository:

1. Create a `.gitignore` file with the below contents (`.idea` is for IntelliJ. Ignore the relevant metadata files of your IDE):

<script src="https://gist.github.com/andrewnessinjim/1c62f0b421e5d26115508013ff354a86.js"></script>

2. Run `git init`
3. Run `git add .`
4. Run `git commit -m "Initial commit"`

## Step 5: Deploy to Heroku

1. Head over to [Heroku's sign up page][9] and create a free account.
2. Follow the [official docs to install][10] the heroku command line utility.
3. `cd` in to your project directory:
    	
`cd mongo-heroku-webapp`
    
4. Run `heroku login` to login to your account.
5. Run `heroku create mongo-heroku-webapp` to create your heroku application based on your current directory.
6. The previous step detects that you have initialized a git repository and sets up the necessary git remotes for deployment. You can verify this with the below command:

<script src="https://gist.github.com/andrewnessinjim/2d51b55fb066adbdefbd1832cba66b77.js"></script>

7. Run `git push heroku master` to deploy your application to heroku.
8. Run `heroku open` to open your application in your web browser. This will reveal your application's URL in the browser. It should look something like this:

`https://mongo-heroku-webapp.herokuapp.com/`
        
Heroku doesn't know the actual URL's we're listening to from our application. But we have the base URL of our application. We can see in the `MyResource` class that we're listening for `/myresource` URL:

`@Path("myresource")`
        
Hence `https://mongo-heroku-webapp.herokuapp.com/myresource` is the full URL to our application. Accessing this URL from the browser should fetch you the message that we have saved in our atlas cluster. :)

![Final output in a browser]({{ site.baseurl }}/images/hello-mongo-heroku/output.PNG)

You might have noticed that we've hardcoded the DB URL, user name and password in Java. This is a bad practice and we can use [config vars in heroku][11] to avoid this. I will create a separate post to explain the steps.

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
[8]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
[9]: https://signup.heroku.com/dc
[10]: https://devcenter.heroku.com/articles/getting-started-with-java#set-up
[11]: https://devcenter.heroku.com/articles/getting-started-with-java#define-config-vars
[12]: https://jersey.github.io/
[13]: https://github.com/andrewnessinjim/mongo-heroku-webapp