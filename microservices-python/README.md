# Microservices Python

Microservice Architecture and System Design with Python & Kubernetes

### Pre-requsities
1. Python3
2. Docker for Windows/Mac
3. Minikube
4. K9s
5. MySQL

After installing MySQL on your local system, use a command to access the database,

```
$ mysql -uroot
```

**Note:** In production enviroment you can secure your database using command `mysql_secure_installation` and follow all the best security practices.

For each micro-service you use a diferent virtualenv so it will not conflict the packages and libraries needed for that micro-service.

### Creating a virtual env

Create a virtualenv,
```
$ python3 -m venv service_name/venv
```

Change a directory to the microservice name,

```
$ cd  service_name
```

Activate the virtualenv,

```
$ source venv/bin/activate
```

To validate the activated virtualenv,

```
(venv) mohanpawar@MohanPawars-MacBook-Pro auth % env | grep VIRTUAL
VIRTUAL_ENV=/Users/mohanpawar/Desktop/workspace/GitHub/python-flask-tutorials/auth/venv
VIRTUAL_ENV_PROMPT=(venv)
```

To exit from the virtualenv,

```
$ deactivate
```

### Auth Service

1. Activcate the virtual env.

2. Install the python3 dependencies,

```
$ pip3 install pylint
$ pip3 install jedi
$ pip3 install pyjwt
$ pip3 install flask
$ pip3 install flask_mysqldb

```

3. Write a code for auth service in `server.py` file.

4. Export the environment variable `MYSQL_HOST`,

```
$ export MYSQL_HOST=localhost
```

5. Create `init.sql` file for creating database and table in the MySQL. Also, insert one entry(email and password) in the table. These credentials will be use to access the database via the auth service.

6. Before executing `init.sql` script, you can check that auth database is not present in the MySQL. Then, execute the `init.sql` file,

```
$ mysql -uroot < init.sql
```

Note: If the user is alredy present then before executing the above script, delete the user and try again. Use the following command to delete the database and user,

```
$ mysql -uroot -e "DROP DATABASE auth"
$ mysql -uroot -e "DROP USER auth_user@localhost"
```

7. After executing `init.sql` script, check and validate that the database auth and table user is created successfully. 

### Gateway Service

### Notification Service

### MP3 Converter Service

### RabbitMQ Producer-Consumer Service