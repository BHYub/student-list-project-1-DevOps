# student-list

This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)

---

## Context

_POZOS_ is an IT company located in France and develops software for High School.

The innovation department want to disrupt the existing infrastructure to ensure that

it can be scalable, easily deployed with a maximum of automation.

POZOS wants to build a "**POC**" to show how docker can help and how much this technology is efficient.

For this POC, POZOS will give us an application and want us to build a "decouple" infrastructure based on "**Docker**".

Currently, the application is running on a single server with any scalability and any high availability.

When POZOS needs to deploy a new release, every time some goes wrong.

In conclusion, POZOS needs agility on its software farm.

## Infrastructure

For this POC, I will only use one single machine with a docker installed on it.

The build and the deployment will be made on this machine.

POZOS recommends to use centos7.6 OS because it's the most used in the company.

## Application

The application that we will be working on is named "_student_list_", this application is very basic and enables POZOS to show the list of the student with their age.

student_list has two modules:

- the first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students

The task is to build one container for each module an make them interact with each other

Application source code can be found [here](https://github.com/diranetafen/student-list.git "here")

The files that are provided (in the delivery) are **_Dockerfile_** and **_docker-compose.yml_**

Now it is time to explain you each file's role:

- docker-compose.yml: to launch the application (API and web app)
- Dockerfile: the file that will be used to build the API image (details will be given)
- requirements.txt: contains all the packages to be installed to run the application
- student_age.json: contain student name with age on JSON format
- student_age.py: contains the source code of the API in python
- index.php: PHP  page where end-user will be connected to interact with the service to - list students with age.

```bash

 $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';
```

## Build of the API image with Dockerfile and test

- Base image

"python:3.8-buster"
But this base image is too heavy so I decided to go with the slim version which is lighter
"python:3.8-slim-buster"
Going from 1.1 Go to 656 with later installs
But using this version, pip was not working properly so I had update pip and run a couple of intalls in the dockerfile

```bash
apt install build-essential libatlas-base-dev -y
```

in addition of what i will install later

- Add the source code

I copied the source code of the API in the container at the root "/" path

- Prerequisite

The API is using FLASK engine,  so I installed some package

```bash
apt update -y && apt install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
```

I copied the requirements.txt file into the container in the root "/" directory to install the packages needed to start up our application

to launch the installation, I used this

```bash
pip3 install -r /requirements.txt
```

But with all these installs we get an image of 656 Mo as said before , so i did some cleaning out there with these commands

```bash
rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && rm -rf /root/.cache/pip \
    && apt-get purge -y --auto-remove build-essential \
    && rm -rf /requirements.txt
```

And we can get an image of 358 Mb from the initial 1.1Go
![Initial Image](/images/Image%20api.PNG)
to
![Final Image](/images/api%20opti.PNG)

- Persistent data (volume)

I created data folder at the root "/" where data will be stored and declare it as a volume

I will use this folder to mount student list

- API Port

To interact with this API I exposed 5000 port

- CMD

When container start, it must run the student_age.py , so it should be something like

```bash
CMD [ "python3", "./student_age.py" ]
```

So Dockerfile is finished and I have to build the image and test it

Running the container and testing it
![running and testing the API](/images/container%20api%20created.PNG)

It's running, nice!

## Infrastructure As Code (5 points)

After testing your API image, I need to put all together and deploy it, using docker-compose.

The **_docker-compose.yml_** file will deploy two services :

- website: the end-user interface with the following characteristics
  - image: php:apache
      - environment: USERNAME and PASSWORD
  - volumes: to avoid php:apache image run with the default website.
  - depend on: I need to make sure that the API will start first before the website
  - port: exposing the port
- API: the image builded before should be used with the following specification
  - image: api_student_age
      - volumes: I did mount student_age.json file in /data/student_age.json
  - port: exposing the port
  - networks: appnet

Runing your docker-compose.yml

![Docker Compose Run](/images/compose%20up.PNG)

Finally, I reach the website and click on the bouton "List Student"

And It's alive !!! ... I mean it works... !!!
![Website working page](/images/final%20app.PNG)

## Docker Registry

POZOS need a private registry and store the built images

I did deploy a private registry to push the api image on:
I used these commands

```bash
#for backend
docker run -d -p 5050:5050 -e REGISTRY_STORAGE_DELETE_ENABLED=true -e REGISTRY_HTTP_HEADERS_Access-Control-Allow-Methods=[HEAD,GET,OPTIONS,DELETE] -e  REGISTRY_HTTP_HEADERS_Access-Control-Credentials=[true] -e REGISTRY_HTTP_HEADERS_Access-Control-Allow-Headers=[Authorization,Accept,Cache-Control] -e REGISTRY_HTTP_HEADERS_Access-Control-Expose-Headers=[Docker-Content-Digest] --net appnet --name registry-eazy registry:2
#for frontend
docker run -d -p 8090:80 -e NGINX_PROXY_PASS_URL=http://registry-eazy:5050 --net appnet -e DELETE_IMAGES=true -e REGISTRY_TITLE=Reg-training --name frontend-eazy joxit/docker-registry-ui:2

```

![Pushed image](/images/registry.PNG)

Nice ! now lets deliver the work to ***eazytrainingfr@gmail.com***

![task done](/images/done.jpg)
