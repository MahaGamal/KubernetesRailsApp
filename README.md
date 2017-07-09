== README

**Rails application running with Kubernetes Using the Minikube

*Prerequisites
 install docker, Minikube and kubectl

 starting 
  minikube start 
  eval $(minikube docker-env)

*Building our Rails Images

  using the rails application that can be found here [https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application]. follow along. 

 after follow, let’s build the image and give it a [drkiq, sidekiq] name
 
  `docker build -t drkiq:1 .`
  `docker build -t sidekiq:1 .`
  

Creating the env ConfigMap

	``` kubectl create configmap env-config \
	--from-literal=postgres_user=drkiq \
	--from-literal=postgres_password=yourpassword \
	--from-literal=postgres_host=postgres \
	--from-literal=WORKER_PROCESSES=1 \
	--from-literal=LISTEN_ON=0.0.0.0:8000 \
	--from-literal=DATABASE_URL=postgresql://drkiq:yourpassword@postgres:5432/drkiq?encoding=utf8&pool=5&timeout=5000 \
	--from-literal=CACHE_URL=redis://redis:6379/0 \
 	--from-literal=JOB_WORKER_URL=redis://redis:6379/0 ```
 to check configration
  kubectl get configmap env-config -o yaml
 to edit 
  kubectl edit configmap env-config

Creating a Database Deployment
 Creating the postgres service
   kubectl create -f deployments/postgres.yml
 Creating the redis service
   kubectl create -f deployments/redis.yml

Creating a app Deployment
  
  kubectl create -f deployments/drkiq.yml
  kubectl create -f deployments/sidekiq.yml


Initialize the Database
kubectl exec [podid] -it -- rake db:reset
kubectl exec [podid] -it -- rake db:migrate

start service
minikube service drkiq --url
