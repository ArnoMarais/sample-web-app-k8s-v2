Arno Marais
Belt Exam 2 – Docker and Kubernetes
URLS
Docker Hub: https://hub.docker.com/repositories/arnomarais
GitHub: https://github.com/ArnoMarais/sample-web-app-k8s-v2

Web Application Introduction

This sample application has a backend application built-in “.Net core”. A frontend application built with HTML, CSS and JavaScript and the database used Postgres.

Application link - https://github.com/saurabhd2106/sample-web-app-k8s.git

The backend application code can be accessed under “api” folder.
The frontend application code can be accessed under “ui” folder.

Question 1 –

1.	Write a Dockerfile to create a Docker image of the backend application and push it to the Docker registry (DockerHub or ECR). 
Dockerfile
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/69b1e17f-f4f4-41ae-881c-c00d95f52baf)

Run the command to build the docker image:
docker build -t basic-3tier-api .
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/dfdbe3d6-bfe2-4025-87cd-66da26c10453)

Run docker image ls to see if the image is build

After the image is created, push the image to AWS ECR you created in your AWS account and region of choice. This will require your AWS account id and AWS access key to login before you push.

sudo aws ecr get-login-password --region ap-southeast-2 | sudo docker login --username AWS --password-stdin 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr
sudo docker tag basic3tier-api 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr:basic3tier-api
sudo docker push 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr:basic3tier-api
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/61bcb52d-2858-48a7-94d9-f13164f7769e)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/720fb51f-8f87-4754-b144-f8c79fde12ae)

2.	Write a Dockerfile to create a Docker image of the frontend application. Use a server like “Nginx” as a base image to create this image. Push this image to the Docker registry.
Dockerfile
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/95f02bca-e215-44c9-94f6-21904d98c65f)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/a549d5b2-bfa6-49bb-8118-c94302e8aef1)
 
Follow the same steps to push to AWS ECR
sudo docker images
sudo aws ecr get-login-password --region ap-southeast-2 | sudo docker login --username AWS --password-stdin 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr
sudo docker tag basic3tier-ui 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr:basic3tier-ui
sudo docker push 517344335201.dkr.ecr.ap-southeast-2.amazonaws.com/codingdojo_ecr:basic3tier-ui  
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/3ac8c0fd-5a3c-4964-9f73-c15fab319e5d)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/f7cf5104-94d2-495a-9d89-ce989660d80e)

3.	Deploy a database container (using postgres:latest image). 
Use the below environment variables for this database.
a.	POSTGRES_USER: postgres
b.	POSTGRES_PASSWORD: admin123
c.	POSTGRES_DB: basic3tier
Make sure to use external volume to store the data.
There are two ways to approach this:
One by using a docker file. By using a Dockerfile, you kan keep the user name and password in the file and pass them in.

The other way is to run
docker run -d -p 5432:5432 -e POSTGRES_USER="postgres" -e POSTGRES_PASSWORD="admin123" -e POSTGRES_DB="basic3tier" --network=back-tier -v postgres-data:/var/lib/postgresql/data --name=basic-3tier-postgres postgres:latest

Dockerfile and Build the postgres DB image 
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/d0e6e8c2-4220-42dc-9217-6fe554d241aa)

Now create the external volume
docker volume create postgres-data
Now run the DB container and mount the volume
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/72fdf815-8094-4281-81d1-aa0d8c4ec7dd)

Did both methods but ended up using 
docker run -d -p 5432:5432 -e POSTGRES_USER="postgres" -e POSTGRES_PASSWORD="admin123" -e POSTGRES_DB="basic3tier" --network=back-tier -v postgres-data:/var/lib/postgresql/data --name=basic-3tier-postgres postgres:latest

4.	Deploy pgadmin (dpage/pgadmin4:latest) for connecting to the above Postgres database.
Run:
docker run --name my-pgadmin -p 82:80 -e 'PGADMIN_DEFAULT_EMAIL=arno.marais2@gmail.com' -e 'PGADMIN_DEFAULT_PASSWORD=admin123' -d dpage/pgadmin4
   ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/1e31e8af-630c-4763-9a60-6158022567ce)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/81f3a61e-b19c-4c8b-953a-c3dfb6436244)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/f8368a9c-8a8d-442c-a57d-08b4a6d2fb39)
 
You will need to run:
docker inspect 492f5d4d32a6
to get the IP of the postgres db.
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/0b4fc66e-2994-4770-8352-3650df7d5c0d)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/219751ff-2e83-425e-9d9f-60d931e09810)

5.	Deploy the frontend application as a docker container running on port 80
docker run -d -p 80:80 --network=front-tier --name frontendui basic3tier-ui
   ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/b128ff9a-1f06-4036-80a6-6c3121223372)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/20b96ed3-5559-47b0-ae14-58a7cba91005)

6.	Deploy the backend application as a docker container running on port 81
docker run -d -p 81:80 --network=back-tier --name backendapi basic3tier-api
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/642888a5-88c1-4454-979d-515847b9450a)

7.	Create the required networks and volumes.
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/25a8a0ef-c5e5-4334-9dee-49f46bd4f485)

docker network connect back-tier backendapi
docker network connect back-tier my-pgadmin
docker network connect back-tier basic-3tier-postgres
docker network connect front-tier frontendui
8.	Verify that your application is up and running.
 ![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/7a28fe82-69de-4dce-a52f-5b83a40119b5)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/dc624e8e-4e64-4916-b85b-b1d9fb63f9fb)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/c498d165-58f1-42c2-a324-10cafa53491b) 

9.	Perform the CRUD operations to verify all three services are communicating with each other.
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/958402f4-a8dd-4feb-9c80-8fd764d291bc)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/0c025c99-416e-42a9-a2fc-26ced63036ac)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/da4767b0-be95-4961-97c1-ad30a807e5b2)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/6478ea40-61b4-46ee-b9a9-089befb3ed2c)

Note – To connect the backend with the frontend, you will need a connection string. 
ConnectionStrings__Basic3Tier: "Host=basic-3tier-postgres;Port=5432;Database=basic3tier;Username=postgres;Password=admin123"
 
Question 2 – 
I used the new revised code.

Deploy the whole application using Docker-compose.
Api/Dockerfile
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/6332a07d-cd3d-4e73-9eb7-534c25202a9e) 

UI/Dockerfile
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/b39fbf16-5242-4a40-852f-3b1e5c185e43)

Docker-compose.yml
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/947bb084-8e72-4cdc-b974-98c2d90ac6b0)

Run docker-compose to create every thing.  
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/45bbf2dd-afbb-40bb-aaba-0f9c40bbc9ef)

Run docker-compose ps to see if everything has created.  
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/645f23f9-b5c7-48bb-b02b-31e5f75e3c5d)

Check that every thing is running.  
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/d6ede2ff-2973-426b-a042-9abb7e0c594b)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/cb996d48-e493-432c-ad74-a7371103589f)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/6184a1be-04bf-474d-a7f7-f120faf4a3eb)

Register the db on pgadmin:
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/73e4ad4b-88a9-4d5a-8fac-993a0f68b5c2)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/664a6fa7-baef-4644-9d63-ecc53fddfb2e)
   
Perform CRUD.      
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/4331b467-c0df-40da-9805-1d95e0aa7cb9)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/b8c0faba-22a0-4048-9ede-069627b322d5)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/c66b2870-1cb6-491a-bc45-116015022ecf)

 
Question 3 –

Deploy the whole application using kubernetes (You can use K3S or minikube or EKS to deploy the application)

1.	Create three deployment yaml files, one each for frontend(name basic3tier-ui), backend (name basic3tier-api), and database(name basic3tier-db).
Frontend-deployment.yml 
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/89736de4-0782-4e59-a524-5f90cbd03332)

Backend-deployment.yml 
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/a72dde5d-bb75-4cfc-ab58-d592049c5e56)

Database-deployment.yml 
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/fb63992f-a2ef-4989-9a62-85d778f8b433)

2.	Deploy one pod of frontend, three pods of backend and one pod of database.
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/4d745c4b-dd7b-44ce-b4a0-1cfb9b57ca5c)
 
3.	Create the required Service to expose “basic3tier-ui” (NodePort)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/0faa0849-2f5d-4d37-bb72-0bb4fa7b9533)
 
4.	Create the required Service to expose “basic3tier-api” (NodePort)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/4449e3b1-f764-4010-88ab-cf534eacb841)
 
5.	Create a required Service to connect basic3tier-ui and basic3tier-api applications (ClusterIP)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/dc4d9c0a-e125-499c-9ca0-fe80f0ee814b)
 
6.	Create a required Service to connect the basic3tier-api with basic3tier-db. (ClusterIP)
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/2e5f0a27-91e2-4ffc-99a2-310fcdb89d12)
 
7.	Create a Nginx-Ingress to expose the basic3tier-ui application. 
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/6f159af6-9418-42cd-85ac-a3a1d8e5a29c)
 
8.	Show the application running and perform some CRUD operations to verify the networking.
![image](https://github.com/ArnoMarais/sample-web-app-k8s-v2/assets/22773627/994e275b-0191-493d-85d2-d3bd6922b46c)
 


