# Учбовий проект по K8S, на основі [курсу](https://www.udemy.com/course/kubernetes-ru/learn/lecture/41373912#overview)


## Конспект K8S


### [Minikube](https://minikube.sigs.k8s.io/docs/start/)


`minikube start`			- старт kubernets
	`--driver=docker` 	- по умолчанию docker

https://minikube.sigs.k8s.io/docs/drivers/kvm2/


`minikube start --driver=kvm2 --container-runtime=containerd` 		- запуск на отличной от докера виртуальной машине

`sudo ctr -n k8s.io containers list` 									- вызов внутри KVM, вместо `docker ps`


`minikube status` 		- статус

`minikube stop` 		- остановить minikube

`minikube delete` 		- удалить minikube

`minikube ip`			- посмотреть IP кластера

`minikube ssh` 			- зайти внутрь кластера

`minikube tunnel` 		- создаем туннель для работы лоадбалансером, чтоб нам выдали IP из интронет, на винде и маке будет 127.0.0.1, на линуксе что вроде 10.109.201.5 


`minikube dashboard`	- запускаем dashboard minikube


------------

### [kubectl](https://kubernetes.io/docs/tasks/tools/)


`kubectl cluster-info`						- показывает пути по которым доступен статус кластера

`alias k=kubectl`							- делаем алиас для удобства

`alias kubectl="minikube kubectl --"` 		- для встроенного kubectl в minikube

`k get namespaces`							- посмотерть доступные пространства имен


------------

#### Pods

`k run <имя пода> --image=<docker образ>`		- создание пода

`k get pods` 									- посмотреть поды
		`--namespace=kube-system`				- по конкретному namespace
		`-o wide`								- более подробная информация

`k describe pod <имя пода>` 					- детали pod


`k delete pod <имя пода>`						- удаление pod



------------

#### Deployment

можно **deployment** или **deploy**


`k create deployment <имя deployment> --image=<docker образ>` 		- создание деплоймента, 

`k describe deploy <имя deployment>` 									- посмотреть детали

`k get deploy` 														- посмотреть deployments

`k scale deploy <имя deployment> --replicas=3` 						- изменяем количество подов (реплик)


------------
#### Services

Можно svc - это короткое название services

`k get services`						- посмотреть существующие сервисы, 

`k expose deploy <имя деплоя>`  		- создание сервиса для деплоя
		`--type=NodePort --port=8080 --target-port=80` 		

				`--port=<порт>`				- открываем наружный порт
				`--target-port=<порт>`		- порт внутренний
				`--type=<тип сервиса>`		- тип сервиса, может быть ClusterIP, NodePort, LoadBalancer. По умолчанию ClusterIP
				`--url` 						- флаг, который позволяет создать туннель к сервису


`k describe service <имя деплоя>` 	- детали сервиса, по умолчанию системный "kubernetes"


`k delete svc <имя деплоя>`			- удлаение сервиса



------------
#### Docker Hub

`docker build . -t ymnik13/k8s-web-hello-ua:latest -t ymnik13/k8s-web-hello-ua:2.0.0`

`docker push <имя пользователя/имя образа>`	 	- заливаем наш образ на docker hub
		`ymnik13/k8s-web-hello-ua --all-tags`

		перел публикацией регаемся на docker hub (docker login). 
		Если будут ошибки то надо сбросить токен доступа для этого виконати `gpg --generate-keyі` тоді `pass init "<user-id - пользователь>"` і потім `docker login`

		For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/


`k set image deploy <имя деплоя> <имя образа>=<новый образ>` 			 				- замена образа
`k set image deploy k8s-web-hello k8s-web-hello-ua=ymnik13/k8s-web-hello-ua:2.0.0`

`k rollout status deploy <имя деплоя>` 												- просмотреть статус изменений

------------

### K8S YAML

`k apply -f deployment.yaml`					- запускаем\обновляем деплой\сервис при помощи yaml

`k delete -f deployment.yaml -f service.yaml` 	- удаление ресурсов по файлу


https://kubernetes.io/docs/reference/using-api/
Kubernetes > Documentation > Reference > API Overview  							- описание API k8s

https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/
Kubernetes > Documentation > Reference > Kubernetes API > Workload Resources	- описание как создавать ресурсы


#####################    deployment.yaml    #####################
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <имя деплоя> 						# содержит название деплоймента
spec:
  selector:
    matchLabels:
      app: <имя деплоя чаще всего, но можно и свое, по нем будет привязка>
  template:
    metadata:
      labels:
        app: <имя деплоя чаще всего но можно и свое>
    spec:
      containers:
      - name: <имя деплоя чаще всего но можно и свое>
        image: <имя образа>					# указываем образ для контейнера
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:								# это чисто для информации, не влияет на реальное открытие 
        - containerPort: <внутренний>		# декларируем какой порт внутри контейнера, 
        - containerPort: 3001				# можно так указать больше портов
```


#####################    service.yaml    #####################
```yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-web-hello 						# имя сервиса
spec:
  type: LoadBalancer 						# тип сервиса
  selector:
    app: k8s-web-hello 						# привязка к деплою
  ports:
    - port: 3333 							# откроет внешний порт
      targetPort: 3000 						# указываем внутренний порт
```



