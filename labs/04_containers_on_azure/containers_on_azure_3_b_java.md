## Exercise 3.b: **Optional**: Multi Stage Build for Java.

For building and packaging a Java app with Maven, we obviously need Maven installed. The base image we used in the preceding exercise only has the runtime, not Maven, thus we need another base image. Additionally, we need to execute the build commands right within the container, which we can achieve using the `RUN` instruction. Yet the resulting image would not be what we would like to run in production. It would not only contain the runtime and our app, it would carry the Maven installation and its dependencies as well and still contains the complete source code of the application. Running this image in production would be like shipping the factory along with the car, containing all construction plans for the car as well.
    
    The additional files and packages in the image not only make it larger and thus less quick to deploy and start, they as well make the image less secure by providing additional attack surface, an attacker might exploit to get access to our application. This needs to be changed.

1. To solve the dilemma that we want to build within the image but we do not want the build tools and source code in the image, we use a concept called Multi Stage Builds. This is achieved with the `AS` keyword that allows us to define a name for an image. This image can then be referenced in additional images defined in the same Dockerfile to copy files out of it with the `--from` flag.
    
    Change your dockerfile to look like this:
    ```Dockerfile
    # Stage 1: Build the app
    FROM maven:3.3-jdk-8 AS builder
    WORKDIR /usr/src/myapp
    COPY myapp .
    RUN mvn clean install
    
    # Stage 2: Build the production image
    FROM tomcat:alpine
    WORKDIR /usr/local/tomcat/webapps
    COPY --from=builder /usr/src/myapp/target/myapp.war .
    RUN mv myapp.war ROOT.war
    RUN rm -rf ROOT
    CMD ["catalina.sh", "run"]
    ```
1. Build and run the container image again with:

    ```sh
    docker image build --tag myappimage .
    docker container rm -f myapp
    docker container run --name myapp -d -p 80:8080 myappimage 
    ```

The second stage in the last Dockerfile produces our production image, which is based on `tomcat` again, so that we have a tiny and more secure resulting containerized application again, but still running the build in a container as well.

[**Click here for the next exercise**](containers_on_azure.md#exercise4)