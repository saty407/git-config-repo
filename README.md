Kafka Debezium

Go to the location of the file you have downloaded and run below commands
Command: docker-compose -f docker-compose.yml up -d (run in background)
to stop the docker command: docker-compose down


This yaml file has the one kafka instance, one zookeeper instance, mysql instance and kafka connect
When we run the command “docker-compose -f docker-compose.yml up -d”
There will be docker instances running all kafka,zookeeper,mysql,kafka connect on same instance
Once docker instance are up, we need to connect the debizum through kafka connect.
Before connection debizum you have two options.
1.	You can dump the existing mysql database “- ./datadump:/docker-entrypoint-initdb.d”
2.	Or you can create you own database by 
a.	Open mysql docker cli and run mysql -uroot -proot
b.	CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'password';
c.	GRANT ALL PRIVILEGES ON * . * TO 'new_user'@'localhost';
d.	FLUSH PRIVILEGES;
e.	Exit;
f.	Again login with the new user and password
g.	Once its accessed create a new database and create table and use that creadentials in the debizum connect with user name, password, table 

Important that you should connect to mysql outside by mysql workbench/any tool so that its accessable to outside the docker once connected use the same details in the debezium connect





version: '3'
services:
    zookeeper_instance_1:
        container_name: zookeeper_instance_1
        image: quay.io/debezium/zookeeper
        ports:
            - 12181:2181
            - 12888:2888
            - 13888:3888
    zookeeper_instance_2:
        container_name: zookeeper_instance_2
        image: debezium/zookeeper
        ports:
            - 12182:2181
            - 12889:2888
            - 13889:3888
    kafka_instance_1:
        container_name: kafka_instance_1
        image: quay.io/debezium/kafka
        environment:
            ZOOKEEPER_CONNECT: zookeeper_instance_1:2181,zookeeper_instance_2:2181
            BROKER_ID: 1
            KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka_instance_1:9091,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19091
            KAFKA_LISTENERS: INTERNAL://kafka_instance_1:9091,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19091
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
        depends_on:
            - zookeeper_instance_1
            - zookeeper_instance_2
        ports:
            - 9091:9091
    kafka_instance_2:
        container_name: kafka_instance_2
        image: quay.io/debezium/kafka
        environment:
            ZOOKEEPER_CONNECT: zookeeper_instance_1:2181,zookeeper_instance_2:2181
            BROKER_ID: 2
            KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka_instance_2:9092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19092
            KAFKA_LISTENERS: INTERNAL://kafka_instance_2:9092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
        depends_on:
            - zookeeper_instance_1
            - zookeeper_instance_2
        ports:
            - 9092:9092
    kafka_instance_3:
        container_name: kafka_instance_3
        image: quay.io/debezium/kafka
        environment:
            ZOOKEEPER_CONNECT: zookeeper_instance_1:2181,zookeeper_instance_2:2181
            BROKER_ID: 3
            KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka_instance_3:9093,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19093
            KAFKA_LISTENERS: INTERNAL://kafka_instance_3:9093,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:19093
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
        depends_on:
            - zookeeper_instance_1
            - zookeeper_instance_2
        ports:
            - 9093:9093
    mysql:
        container_name: mysql
        environment:
            MYSQL_PASSWORD: mysqlpw
            MYSQL_ROOT_PASSWORD: debezium
            MYSQL_USER: mysqluser
        image: mysql
        ports:
            - 3307:3306
            
    connect:
        container_name: connect
        environment:
            OFFSET_STORAGE_TOPIC: my_connect_offsets
            STATUS_STORAGE_TOPIC: my_connect_statuses
            CONFIG_STORAGE_TOPIC: my_connect_configs
            GROUP_ID: '1'
            BOOTSTRAP_SERVERS: kafka_instance_1:9091,kafka_instance_2:9092,kafka_instance_3:9093
        image: quay.io/debezium/connect
        depends_on:
            - zookeeper_instance_1
            - zookeeper_instance_2
            - kafka_instance_1
            - kafka_instance_2
            - kafka_instance_3
            - mysql
        ports:
            - 8083:8083
    kafdrop:
        container_name: kafdrop
        environment:
            KAFKA_BROKERCONNECT: kafka_instance_1:9091,kafka_instance_2:9092,kafka_instance_3:9093
        image: obsidiandynamics/kafdrop
        ports:
            - 9000:9000

# http://localhost:9000/topic/dbserver1.inventory.addresses
# http://localhost:8083/connectors

# Postman
# curl --location --request POST 'http://localhost:8083/connectors' \
# --header 'Content-Type: application/json' \
# --data-raw '{
#  "name": "inventory-connector",  
#  "config": {  
#    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
#    "tasks.max": "1",  // fixed
#    "database.hostname": "mysql",   // this is from 
#    "database.port": "3306",   // this is always same
#    "database.user": "debezium", // user name when creating new db and 
#    "database.password": "dbz", // same password you provide while creating new db
#    "database.server.id": "1",  // unique value
#    "database.server.name": "dbserver1",  // any unique name
#    "database.include.list": "inventory",  // database name created 
#    "database.history.kafka.bootstrap.servers": "kafka_instance_1:9091,kafka_instance_2:9092,kafka_instance_3:9093",  
#    "database.history.kafka.topic": "schema-changes.inventory"  // unique name
#  }
# }'
# }' 
