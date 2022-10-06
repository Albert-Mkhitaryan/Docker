# Docker
## Pulling/Pushing Docker images
An [image](https://docs.docker.com/get-started/overview/#:~:text=of%20those%20objects.-,Images,-An%20image%20is) is a read-only template with instructions for creating a Docker container

Pulling from public docker hub registry
```sh
docker pull nginx
```
By default, the image with the "latest" tag will be pulled. For pulling other versions we need to provide an appropriate tag.
```sh
docker pull nginx:1.23.1
```
If we have a private repository we need to login into that repository in order to pull/push images. for example, If I try to pull an image from my private repo without logging in it will fail.
```sh
docker pull albertmkhitaryan/demo:test_nginx
```
Now lets try pulling the image after login
```sh
docker login --help 
docker login --username albertmkhitaryan
docker pull albertmkhitaryan/demo:test_nginx
```
For pushing to private repository we need to tag our image appropriately. \<remorisotyname:tag\>
```sh
docker tag nginx:latest albertmkhitaryan/demo:test_nginx_new
docker push albertmkhitaryan/demo:test_nginx_new
docker login --username albertmkhitaryan
docker push albertmkhitaryan/demo:test_nginx_new
```

## Running docker Containers
A [container](https://docs.docker.com/get-started/overview/#:~:text=other%20virtualization%20technologies.-,Containers,-A%20container%20is) is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI.
Lets run our nginx image which we have pulled earlyer
```sh
docker images 
docker run nginx 
```
Run container in detached mode with environment variables.
```sh
docker run -d  --name testing \
-e MYSQL_ROOT_PASSWORD=my-password \
-e MYSQL_HOST=% \
mysql:latest 
docker ps 
```
  - -d runs container in detached mode
  - --name starts a container with a predefined name, it makes easier interactions with container
  - -e set environment variables for running instance

Run commands in running container 
```sh
docker ps 
docker exec -it testing mysql -h 127.0.0.1 -u root -pmy-password
docker exec -it testing bash 
```
 - exec help us to execute commands on running container
 - -it: -t for getting tty "terminal", -i for interactive

Expose Ports 
```sh
docker run -d  --name testing_ports \
-e MYSQL_ROOT_PASSWORD=my-password \
-e MYSQL_HOST=% \
-p 33006:3306 \
mysql:latest

netstat -nltp  | grep 33006 
mysql -h 192.168.56.101 -P 33006 -u root -pmy-password
```
 - -p mapping ports on Host OS and Container \<host os port: container port \>

Persistent data
```sh
docker run -d  --name testing_volume \
-e MYSQL_ROOT_PASSWORD=my-password \
-e MYSQL_HOST=% \
-p 33006:3306 \
-v $(pwd)/mysql_data:/var/lib/mysql \
mysql:latest
```
 - -v used for volume mounts 

Now let's do the following to be sure that our persistent data will not be deleted
 - list our data directory 
 - stop your container and list data directory
 - remove the container and list data directory one more time 
```sh
ls -l mysql_data 
docker stop testing_volume
ls -l mysql_data 
docker rm testing_volume
ls -l mysql_data
```
Run another container which will use the same data directory.
```sh
docker run -d  --name testing_volume_new \
-p 33006:3306 \
-v $(pwd)/mysql_data:/var/lib/mysql \
mysql:latest
```


## Dockerfile 
Docker can build images automatically by reading the instructions from a Dockerfile. 
```sh
echo "Hi there !!!" > ./my-web/index.html

cat <<EOF > Dockerfile
FROM nginx:1.23.1
COPY ./my-web/index.html /usr/share/nginx/html
EOF

docker build -t nginx:my-web-server -f Dockerfile . 
docker run -d nginx:my-web-server -p 80:80
```

Example of multystage Dockerfile 
```sh
cat <<EOF > Dockerfile_multystage
FROM maven:3.5.0-jdk-8-alpine AS builder
ADD ./pom.xml pom.xml
ADD ./src src/
RUN mvn clean package

# Second stage: minimal runtime environment
From openjdk:8-jre-alpine
# copy jar from the first stage
COPY --from=builder target/my-app-1.0-SNAPSHOT.jar my-app.jar

CMD ["java", "-jar", "my-app.jar"]
EOF
```
More details can be found in https://docs.docker.com/engine/reference/builder/ 

## Docker Compose  
Compose is a tool for defining and running multi-container Docker applications. https://docs.docker.com/compose/
```sh
cat <<EOF > docker-compose.yml
services:
  db:
    image: mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:
EOF
```