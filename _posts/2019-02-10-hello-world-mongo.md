---
published: false
---
# Hello World with Java in Heroku and MongoDB in Atlas
We'll create a java application with a main method that exposes a web service. This service responds with a message fetched from an M0 instance running in [MongoDB Atlas][1]. You can download the code from here. All the tools used are free.

## Prerequisites
[Maven][4]
[JDK][5]

## Step 1: Build and run a jersey hello world application
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

## References
* [Heroku's official "Getting Started on Heroku with Java" tutorial][2]
* [Jersey's official "Creating a Web Application that can be deployed on Heroku" tutorial][3]

[1]: https://www.mongodb.com/cloud/atlas
[2]: https://devcenter.heroku.com/articles/getting-started-with-java
[3]: https://jersey.github.io/documentation/latest/getting-started.html#heroku-webapp
[4]: https://maven.apache.org/install.html
[5]: https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/