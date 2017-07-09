**Rails application running with Kubernetes Using the Minikube**
     
**Goals**:
   1- Create an image for a Rails application
   2- create a ConfigMap to define environment variables 
   3- Create a Database, Cache and Rails services


_Prerequisites_
 install docker, Minikube and kubectl
 
 _start cluster_
  `minikube start` 
  `eval $(minikube docker-env)`

_Building our Rails Images_

  Using the rails application that can be found here [https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application]. follow along. 

 after follow, let’s build the image and give it a [mahaga50/drkiq] name
 
  `docker build -t mahaga50/drkiq:1 .`
  
_Creating the env ConfigMap_

	` kubectl create configmap env-config \
	--from-literal=postgres_user=drkiq \
	--from-literal=postgres_password=yourpassword \
	--from-literal=postgres_host=postgres \
	--from-literal=WORKER_PROCESSES=1 \
	--from-literal=LISTEN_ON=0.0.0.0:8000 \
	--from-literal=DATABASE_URL=postgresql://drkiq:yourpassword@postgres:5432/drkiq?encoding=utf8&pool=5&timeout=5000 \
	--from-literal=CACHE_URL=redis://redis:6379/0 \
 	--from-literal=JOB_WORKER_URL=redis://redis:6379/0 `

 to check configration
  `kubectl get configmap env-config -o yaml`
 if found error you can edit: 
  `kubectl edit configmap env-config`

_Creating a Database and Cache Deployment_

 Creating the postgres service
   `kubectl create -f deploy/postgres.yml`

 Creating the redis service
   `kubectl create -f deploy/redis.yml`

_Creating an app Deployment_
  
  `kubectl create -f deploy/drkiq.yml`
  `kubectl create -f deploy/sidekiq.yml`


_Initialize the Database_

  `kubectl exec [podid] -it -- rake db:reset`
   `kubectl exec [podid] -it -- rake db:migrate`

_start service_
  `minikube service drkiq --url`
