## Locale

Kada radimo sa kubernetisom lokalno moramo da instaliramo <span style="color:red">minikube</span> i njen cli alat <span style="color:red">kubectl</span>

moramo da startujemo minikube u dokeru,ako smo ga skinuli i update je mozemo samo da idemo  u docker i da ga start.Ako nije update mozemo pokrenuti komandu

```
minikube start --driver=docker
```

proverimo status

```
minikube status
```

da vidimo informacije o k8s nodovima koji su se startali

Ovo vraca informacije o klasteru

```
kubectl cluster-info 
```

Ovo vraca informacije o nodovima

```
kubectl get node
```

Da bi nasu spring app pokrenuli uopste moramo da napravimo Dockerfile u spring app i tu dodmo one stvari da bi mogli da dokerizujemo app celu

 Po defaultu ne kubernetes tj minikube ne moze da cita docker containers i to,pa moramo da mu omogucimo

```
minikube docker-env
```

i kopiramo zadnji komandu sa @FOR i to izvrsimo

i sada kada smo to izvrsili imacemo kubernetes images u docker-u

kada smo to zavrsili odemo i napravimo image iz nase spring app(Moramo da izvrsimo navigaciju u spring app folder)

```
docker build -t springboot-k8s-demo:1.0 .
```

Kada smo napravili image mozemo  da je run u pod-u,to radimo preko kubectl komande i referenciramo docker image

kubectl create deployment IMEDEPLOYMENTA --image=IMEIMG:VERSION --port= PORT

```
kubectl create deployment spring-boot-k8s --image=springboot-k8s-demo:1.0 --port=8080
```

Da bi videli da se stvarno pokrenulo idemo:

```
kubectl get deployment
```

```
kubectl describe deployment
```

```
kubectl get pods
```

```
kubectl logs imepoda
```

Sa ovim logovima vidimo da se nasa spring boot app runuje unutar poda,kako da dodjemo do ip-ja toga da mi pristupimo?

Moramo da napravimo Service obj i da expose port

MI uzimamo deployment i njega expose da se naprvi serviec obj

MI kada expose deployment moramo da kazemo kog je tipa jer tu spada i loadbalancer i NodePort(ovo zelimo)

```
kubectl expose deployment DEPLOYMENTNAME --type=NodePort
```

i sada mozemo da vidimo preko

```
kubectl get service
```

Sada da bi mi pristupili ovome moramo da ga otvorimo za javnost preko minikube(Zadnji sloj kao)

```
minikube service SERVICENAMA --url
```

```
minikube dashboard
```



## Minikube with yml

Moramo da kreiramo yml fajl

kind: Deployment

​		Service

selector i labels moraju da budu isti u Service i Deployment

```yml
apiVersion: apps/v1
kind: Deployment # Kubernetes resource kind we are creating
metadata:
  name: spring-boot-k8s
spec:
  selector:
    matchLabels:
      app: spring-boot-k8s
  replicas: 2 # Number of replicas that will be created for this deployment
  template:
    metadata:
      labels:
        app: spring-boot-k8s
    spec:
      containers:
        - name: spring-boot-k8s
          image: springboot-k8s-demo:1.0 # Image that will be used to containers in the cluster
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080 # The port that the container is running on in the cluster

```

i sada ovo mozemo da run preko:

```
kubectl apply -f FILENAME.yml
```



mi smo ovime napravili 2 app koje se run na razlicitim portovima,kako imamo vise moramo da napravimo loadbalancer

to radi sve service jer on expose nase app u spoljni svet

Service pravimo isto u yml fajlu 

```yml
apiVersion: v1 # Kubernetes API version
kind: Service # Kubernetes resource kind we are creating
metadata: # Metadata of the resource kind we are creating
  name: springboot-k8s-svc
spec:
  selector:
    app: spring-boot-k8s
  ports:
    - protocol: "TCP"
      port: 8080 # The port that the service is running on in the cluster
      targetPort: 8080 # The port exposed by the service
  type: NodePort # type of the service.
```

i izvrsimo istu komandu apply'

i da bi uzeli ip da pristupimo,pre smo kreirali service rucno pa je on vratio ip automatski,ovde to nemamo moramo sami da vidimo

```
kubectl get nodes -o wide
```

ipaddress=nodeIp:nodeport

nodeip -> INTERNAL-IP (kubectl get nodes -o wide)

nodeport->PORT ali ovaj drugi port posle / (kubectl get services)

 <span style="color:red">Ako stavljamo sve u isti fajl imamo --- (mora 3 crtice da se odvoji sekcija) </span>

 <span style="color:red">Kada imamo deployment imamo za apiVersion apps/v1 (mora apps) ,kada radimo za service ide samo v1</span>



## Working with DB

Prvo moramo da deploy mysql intsancu u podu pa tek onda nasu app,jer ce tako prvo dati ip od mysql i port gde mozemo da ubacimo u nasu final app

## Horizontal scaling

Ovo  je kada vidimo da se trenutne replike mikroservisa muce da odrade posao,tj da su preopterecene



```
kubectl get hpa
```



```
kubectl autoscale deployment deployment-name --min=3 --max=10 --cpu-percent=70
```

KOntroler manager ce ovime da upravlja sada
