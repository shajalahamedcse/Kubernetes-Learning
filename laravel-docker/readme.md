# Deploying a Laravel Application to Minikube

### Getting started

    $ composer create-project laravel/laravel:~5.7 laravel-docker
    $ cd laravel-docker
    
### Setting Up Docker

    ├── .docker
    │   └── php
    │       ├── Dockerfile
    │       └── vhost.conf
    ├── app
    ├── artisan
    ├── bootstrap
    ├── config
    ├── database
    ├── public
    ├── readme.md
    ├── resources
    ├── routes
    ├── storage
    ├── tests
    ├── composer.json
    ├── composer.lock
    ├── docker-compose.yml
    └── webpack.mix.js
    
    
    $ mkdir .docker/
    $ touch .docker/Dockerfile .docker/vhost.conf
    
    
    
    
### Dockerfile
    
    FROM php:7.1.8-apache
    
    MAINTAINER Shajal Ahamed
    
    COPY . /srv/app
    COPY .docker/vhost.conf /etc/apache2/sites-available/000-default.conf
    
    RUN chown -R www-data:www-data /srv/app && a2enmod rewrite
    
    
    
### The Apache Vhost

    <VirtualHost *:80>
        DocumentRoot /srv/app/public
    
        <Directory "/srv/app/public">
            AllowOverride all
            Require all granted
        </Directory>
    
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    
    
### Building the Image

    $ docker build --file .docker/Dockerfile -t laravel-docker .
    
    
### Running Apache

    $ docker run --rm -p 8080:80 laravel-docker
    
    
### Log into the Docker Hub from the command line

    $ docker login
 
 
### Check the image ID using

    $ docker images
    
   and what you will see will be similar to
   
    REPOSITORY                                                 TAG                 IMAGE ID            CREATED             SIZE
    laravel-docker                                             latest              e3e331ef27c5        3 minutes ago       461MB
    <none>                                                     <none>              9e957e3dea90        4 minutes ago       391MB
    shajalahamedcse/laravel-kubernetes-demo                    latest              b88658379568        8 hours ago         465MB
    
   and tag your image

    docker tag e3e331ef27c5 shajalahamedcse/laravel-docker
    
### Start  Minikube

    $ minikube -p itech start   
    
### Create a Deployment YAML

    $nano deployment.yaml
    
This command will open an nano editor.


    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: helloweb
      labels:
        app: hello
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: hello
          tier: web
      template:
        metadata:
          labels:
            app: hello
            tier: web
        spec:
          containers:
          - name: hello-app
            image: shajalahamedcse/laravel-docker
            ports:
            - containerPort: 80 

Save it .
            
   
     $ kubectl create -f deployment.yaml
     
       deployment.apps/helloweb created
       
       
You can also use the Minikube GUI dashboard to monitor the cluster.

The GUI also helps with visualising most of the discussed concepts.

To view the dashboard, just run the following:

    $ minikube dashboard
    
or to acquire the dashboard's URL address:

    $ minikube dashboard --url=true
    

You can create a service with:

    $ kubectl expose deployment helloweb --type=NodePort --port=80
    
    service "helloweb" exposed
    
    
Running the following command:

    $ kubectl get services
    
A more exciting way to verify this deployment and the service exposure is obviously seeing the running application in the browser 😊

To obtain the URL of the application (service), you can use the following command:

    $ minikube -p itech service --url=true helloweb     
    
    http://192.168.99.105:30260
    
    
or, launch the application directly in the browser:

    $ minikube -p itech service helloweb
    
    
