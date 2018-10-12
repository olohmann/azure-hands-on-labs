## Exercise 3.b: **Optional**: Multi Stage Build for .NET Core

1. For building and packaging a .NET Core app, we need the .NET SDK installed. The base image we used in the preceding exercise only has the runtime, not the SDK, thus we need another base image. Additionally, we need to execute the publish command right within the container, which we can achieve using the `RUN` instruction. Change your Dockerfile to look like this:

   ```Dockerfile
   FROM microsoft/dotnet:2.1-sdk
   WORKDIR /source
   COPY . .
   RUN dotnet publish --output /app --configuration Release
   WORKDIR /app
   ENTRYPOINT ["dotnet", "myapp.dll"]
   ```

1. (Optional) Build and run the container image again with:

    ```sh
    docker image build --tag myappimage .
    docker container rm -f myapp
    docker container run --name myapp -d -p 80:80 myappimage 
    ```
    It should just work as it did before. The only real difference is that we built the whole thing right within our container image and we changed the `WORKDIR` in between.
    
    Yet the resulting image is not what we would like to run in production. It contains not only the runtime and our app, it carries the SDK as well and still contains the complete source code of the application. Running this image in production would be like shipping the factory along with the car, containing all construction plans for the car as well.
    
    The additional files and packages in the image not only make it larger and thus less quick to deploy and start, they as well make the image less secure by providing additional attack surface, an attacker might exploit to get access to our application. This needs to be changed.

1. To solve the dilemma that we want to build within the image but we do not want the build tools and source code in the image, we use a concept called Multi Stage Builds. This is achieved with the `AS` keyword that allows us to define a name for an image. This image can then be referenced in additional images defined in the same Dockerfile to copy files out of it with the `--from` flag.
    
    Change your dockerfile to look like this:
    ```Dockerfile
    # Stage 1
    FROM microsoft/dotnet:2.1-sdk AS  builder
    WORKDIR /source
    COPY . .
    RUN dotnet publish --output package  --configuration Release
    # Stage 2
    FROM  microsoft/dotnet:2.1-aspnetcore-runtime
    WORKDIR /app
    COPY --from=builder /source/package .
    ENTRYPOINT ["dotnet", "myapp.dll"]
    ```
    
1. Build and run the container image again with:

    ```sh
    docker image build --tag myappimage .
    docker container rm -f myapp
    docker container run --name myapp -d -p 80:80 myappimage 
    ```

The second stage in the last Dockerfile produces our production image, which is based on `microsoft/dotnet:2.1-aspnetcore-runtime` again, so that we have a tiny and more secure resulting containerized application again, but still running the build in a container as well.

[**Click here for the next exercise (Exercise 4)**](containers_on_azure.md#exercise4)