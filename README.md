
# Conatiner Based Development
Log in to https://labs.play-with-docker.com/
## Simple App
```sh
docker run --rm -it -v $PWD:/my -w /my maven:3-jdk-9-slim bash
# inside the container
mvn -q archetype:generate # filter: jdk9, item #1, group: grp, artifactId:yourname-jdk
cd yourname-jdk
mvn -q package
java -jar target/yourname_app-1.0-SNAPSHOT.jar
docker build -t test yourname-jdk
docker run --rm test
```

## REST Service
```sh
docker run --rm -it -v $PWD:/my -w /my maven:3-jdk-9-slim mvn -q archetype:generate
# filter: dropwizard-app, item #1, group: grp, artifactId:yourname-dw
cd yourname-dw
docker run --rm -it -v $PWD:/my -w /my maven:3-jdk-9-slim mvn -q package
docker build -t test .
docker run --rm -p 8080:8080 test 
```
### Using docker compose
```sh
docker-compose up -d
# verify the services are up using
docker-compose ps
```
### Add DB
Uncomment db definition in docker-compose.yml. Repeat last step.
Shutdown the cluster using ``docker-compose down``. Launch it again and verify that the data is there.
### Scale
Try scaling using ``docker-compose scale yourname-dw=3``. Doesn't work.

Edit the docker-compose.yml, removing 8081, 3306 ports, and remove mapping for 8080.
```yml
version: "3"
services:
  yourname-dw:
    build: .
    image: yourname-dw:1.0-SNAPSHOT
    ports:
      - "8080"
    environment:
      JDBC_DRIVER: "com.mysql.cj.jdbc.Driver"
      JDBC_URL: "jdbc:mysql://db:3306/mydb?useSSL=false"

  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456
      MYSQL_DATABASE: mydb
    volumes:
      - yourname-dw-volume:/var/lib/mysql

volumes:
  yourname-dw-volum
```

run:
```sh
docker-compose down
docker-compose up -d --scale yourname-dw=3
```
### Swarm
```sh
docker-compose down
docker swarm init --advertise-addr 192.168.0.49
docker deploy -c docker-compose.yml mystack
docker service ls
docker service scale mystack_yourname-dw=3
docker service ls
```

## Kafka Clients
```sh
docker run --rm -it -v $PWD:/my -w /my maven:3-jdk-9-slim mvn -q archetype:generate -Dfilter=kafka-app
cd mykafka/
docker run --rm -it -v $PWD:/my -w /my maven:3-jdk-9-slim mvn -q package
docker-compose build
docker deploy -c docker-compose.yml mystack
watch docker service ls
docker service logs -f mystack_mykafka-consumer
docker service scale mystack_mykafka-consumer=3
docker service logs -f mystack_mykafka-consumer
docker stack rm mystack
```




