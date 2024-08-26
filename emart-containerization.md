# Emart App Microservices Containerization

This is an e-commerce app that has microservices, frontend is using Nginx, has NodeJS, Angular and JAVA as backend services. NodeJS has a NoSQL MongoDB database as its database and Relational MySQL database as its database.

## Emart App

![alt](/img/Emart-App.png)

1. Clone source [code](https://github.com/devopshydclub/emartapp)
2. Open it in [VS Code](https://code.visualstudio.com/)

       code .

3. There are following directories; client, nginx, javaapi and nodeapi
    
    - For Client
       - Go to client > [Dockerfile](./client/Dockerfile)
       - It's a multi-stage Dockerfile
         - First Stage
           - The first image is from node:14 and that gets renamed to web-build
           - Creates a working directory called /usr/src/app
           - Copies all files from source (./ ) to the image (./client)
           - Runs a cd command, nmp install and a build command
         - Second Stage 
           - The second image is created from Nginx:latest 
           - Copies from the web-build images the files to /usr/share/nginx/html folder
           - Copies the nginx.conf file and replaces the default file
           - Exposes the port 4200
   - For Node api
     - Go to nodeapi > [Dockerfile](./nodeapi/Dockerfile)
     - It's a multi-stage Dockerfile
         - First Stage
           - The first image is from node:14 and that gets renamed to nodeapi-build
           - Creates a working directory called /usr/src/app
           - Copies all files from source (./ ) to the image (./nodeapi)
           - Runs a cd command and nmp install
         - Second Stage 
           - The second image is created from node:14
           - Creates a working directory /usr/src/app 
           - Copies from the nodeapi-build images the files to current directory
           - Runs ls
           - Exposes the port 5000
           - CMD runs a shell and go into the directory to start the app
   - For Node api
     - Go to javaapi > [Dockerfile](./javaapi/Dockerfile)
     - It's a multi-stage Dockerfile
         - First Stage
           - The first image is from openjdk:8 and that gets renamed to BUILD-IMAGE
           - Creates a working directory called /usr/src/app
           - Copies all files from source (./ ) to the image (usr/src/app)
           - Runs mvn install to build the artifact and some tests commands
         - Second Stage 
           - The second image is created from openjdk:8
           - Creates a working directory /usr/src/app 
           - Copies from the BUILD-IMAGE image the file to current directory and the artifact gets renamed to book-work-0.0.1.jar
           - Exposes the port 9000
           - Entrypoint will run command "java" with argument "jar", the app named "book-work-0.0.1.jar"
   - For Nginx
     - The image will be build from official image attaching a config file
     - The [default.conf](/emartapp/nginx/default.conf) file has 3 routing rules; "/" for Angular/client container, the "/api" for NodeJS/node container and "/webapi" for Java container

4. The Docker Compose file there are all services listed
   - You have client - Angular running in Nginx container
   - You have api - Node
   - You have webapi - Java
   - You have Nginx api gateway
   - MongoDB service
   - Emart which is MySQL

To write Dockerfile for all the four services depends on understanding Angular, NodeJS and Java in this case. Which may be a mammoth task. This may require working with the developers and how to host the application. For example Angular return html files which can be run with Nginx or Apache, if Java, it can be ran directly on JDK or Tomcat containers, Node, know the build process of an artifact and host it on NodeJS container itself. The key is to know the build process and hosting method. Your job will be writing a Dockerfile and Docker Compose file, this takes some trial and error approach.

5. Build & Run Microservice App

   1. Launch an EC2 Instance, Install Docker Engine & Docker Compose
          - Go to EC2 > Instances > Launch instance
          - Name and tag 
            - DockerEngine
          - Application and OS Images (Amazon Machine Image)
            - Ubuntu Server 22.04 LTS
          - Instance type
            - t3.medium
          - Key pair(login)
            -  Create new key pair
               -  dockerKey
         - Network settings
           - Allow SSH traffic: My IP
           - Allow HTTP traffic from the internet
         - Configure storage
           - 15 GiB
           - gp2
         - Advanced Details
           - Copy and paste [the script](/emartapp/docker-engine.sh) here
         - Hit Launch instance
        
        NOTE: If the script doesn't work you can install manually after SSHing into the instance. Paste the code into the "docker-engine.sh" in step 2

   2. SSH into the Docker Engine instance (locate your key and change permissions)

          # Change permissions & SSH
          chmod "400" dockerKey.pem
          ssh -i "dockerKey.pem" ubuntu@public-IP

          # Create a script and run it
          sudo vim docker-engine.sh
          chmod +x docker-engine.sh
          ./docker-engine.sh
          sudo usermod -aG docker ${USER}
          exit
            
          # Verify Docker Engine group and Docker Compose version  
            id ubuntu
            docker-compose --version
   3. Clone the [source code](https://github.com/devopshydclub/emartapp)
   
          git clone https://github.com/devopshydclub/emartapp
          ls
          cd emartapp
          sudo docker-compose build -d 
   
   4. Verify the images

          docker images
          docker-compose ps 
          docker-compose top
          docker-compose logs container-name

   5. Create and run containers
          
          docker-compose up
          http://paste-public-IP-on-your-browser
   6. Test the app
      1. Register > Enter your details
      2. Log in and check the Book Mart
   7. Once done

          docker-compose down 
          docker rmi image-id


  