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
g.	Once its accessed create a new database and create table and use that creadentials in the debizum connect with user name, password, table name
