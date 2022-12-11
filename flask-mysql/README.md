## Flask MySQL sample application

### Use with Minikube Kubernetes Environments

### Python/Flask application using a MySQL database

Project structure:

```
.
├── Dockerfile
├── README.md
├── flaskapi.py
├── requirements.txt
```

[manifests](kubernetes_config_files.yaml)

### Getting Started with your own kubernetes cluster

The application we will build consists of two micro-services:
1. a MySQL database
2. a Flask app that implements an API to access and perform CRUD (create, read, update delete) operations on the database.                                                                                                                                                     
When setting up MySQL database we need to take into account two things,
1. To access the database we need some credentials configured.
2. We will need a persistent volume for the database so we will not lose all our data if the nodes will accidently be taken down.

### Creating Secrets

Before creating secret, let's create a base64 of our password which will be use for database access.

```
$ echo -n mohan@123 | base64
bW9oYW5AMTIz
```

Now you can create a secret, `flaskapi-secret.yaml`

```
apiVersion: v1
kind: Secret
metadata:
  name: flaskapi-secrets
type: Opaque
data:
  db_root_password: bW9oYW5AMTIz
```

You can now add the secrets to your cluster via your terminal: 

```
kubectl apply -f secrets.yml
```

And see if it worked by checking the secrets via, 

```
kubectl get secrets
```

### Persistent Volume
Making your application use a persistent volume exists of two parts:
1. Specifying the actual storage type, location, size and properties of the volume.
2. Specify a persistent volume claim that requests a specific size and access modes of the persistent volume for your deployments.

Create a file `mysql-pv.yaml` refer it from manifests directorty.

### Deploy the MySQL server

You can now deploy the MySQL server with 

```
kubectl apply -f mysql-deployment.yml
```

And see if a pod is running via, 

```
kubectl get pods
```

### Create database and table

The last thing we have to do before implementing the API is initializing a database and schema on our MySQL server. We can do this using multiple methods, but for the sake of simplicity let’s access the MySQL server via the newly created service. As the pod running the MySQL service is only accessible from inside the cluster, you will start up a temporary pod that serves as mysql-client:

1. Set up the mysql-client via the terminal: 

```
kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql --host mysql --password=bW9oYW5AMTIz
```

Fill in the (decoded) password that you specified in the secrets.yml file.

2. Create the database, table and schema. You can do whatever you like, but to make sure the sample Flask app will work do as follows:

```
CREATE DATABASE flaskapi;
USE flaskapi;
CREATE TABLE users(user_id INT PRIMARY KEY AUTO_INCREMENT, user_name VARCHAR(255), user_email VARCHAR(255), user_password VARCHAR(255));
```

### Deploying the API

Finally it is time to deploy your REST API. The following gist demonstrates an example of a Flask app that implements the API with only two endpoints. One for checking if the API functions and one for creating users in our database. In the GitHub repo you can find the python file that has endpoints for reading, updating and deleting entries in the database as well. The password for connecting to the database API is retrieved from the environment variables that were set by creating secrets. The rest of the environment variables (e.g MYSQL_DATABASE_HOST) is retrieved from the MySQL service that was implemented before (further on I will explain how to make sure the Flask app has access to this information).

Once you complete the code in `flaskapi.py` create a docker image using the Dockerfile.

```
$ docker build -t mohan08p/flask-api .
```

Finally deploy the application into Kubernetes cluster.

### Making Requests to the API

You are now ready to use our API and interact with your database. The last step is to expose the API service to the outside world via your terminal: 

```
minikube service flask-service
```

You will now see something like

```
(venv) mohanpawar@MohanPawars-MacBook-Pro flask-mysql % minikube service flask-service
|-----------|---------------|-------------|---------------------------|
| NAMESPACE |     NAME      | TARGET PORT |            URL            |
|-----------|---------------|-------------|---------------------------|
| default   | flask-service |        5000 | http://192.168.49.2:31546 |
|-----------|---------------|-------------|---------------------------|
🏃  Starting tunnel for service flask-service.
|-----------|---------------|-------------|------------------------|
| NAMESPACE |     NAME      | TARGET PORT |          URL           |
|-----------|---------------|-------------|------------------------|
| default   | flask-service |             | http://127.0.0.1:50116 |
|-----------|---------------|-------------|------------------------|
🎉  Opening service default/flask-service in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

Go to the provided URL and you will see the `Hello World` message, to make sure your API is running correctly. 

You can now interact with the API using your favorite request service like Postman or curl in your terminal. To create a user provide a json file with a name, email and pwd field. 

for example:

```
curl -H "Content-Type: application/json" -d '{"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>"}' <flask-service_URL>/create
```

If you implemented the other methods of the API (as defined in the GitHub repo) as well, you may now be able to query all users in the database via: 

```
curl <flask-service_URL>/users
```
