# Helm

Ovo je package manager za k8s

Omogucava nam da lakse upravljamo k8s yml fajlovima kada kreiramo k8s projekat

Bez ovoga mi moramo da pokrecemo dosta yml fajla preko kubectl-a,nekad cemo imati 100 yml fajla,pa ce to biti problem,moramo sve to odjednom preko `Helm`-a

Kada radimo sa Helm-om mi definisemo template neki i u njega ubacujemo dynamic variables

service-template.yml

```go
{{- define "common.service" -}} ->da ovo od sad zovemo common.service
apiVersion: v1
kind: Service
metadata:
	name: {{ .Values.deploymentLabel }}
spec:
	selector:
		app: {{ .Values.deploymentLabel }}
	type: {{ .Values.service.type }}
	ports:
		- protocol: TCP
		  port: {{ .Values.service.port }}
		  targetPort: {{ .Values.service.targetPort }}
{{- end -}}
```

deployment-template.yml

```go
{{- define "common.deployment" -}}
	apiVersion: apps/v1
	kind: Deployment
	metadata:
		name: {{ .Values.deploymentName }}
		labels:
			apps: {{ .Values.deploymentLabel }}
	spec:
		replicas: {{ .Values.replicaCount }}
		selector:
			matchLabels:
				app: {{ .Values.deploymentLabel }}
			spec:
				containers:
					- name: {{ .Values.deploymentLabel}}
					  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
					  ports:
						- containerPort: {{ .Values.containerPort }}
						  protocol: TCP
						env:
							{{- if .Values.profile_enabled }}
						- name: SPRING_PROFILES_ACTIVE
						  valueFrom:
						  	configMapKeyRef:
						  		name: {{ .Values.global.configMapName }}
						  		key: SPRING_PROFILES_ACTIVE
							
							 {{- end }}
							 {{- if .Values.zipkin_enabled }}
							 - name: SPRING_ZIPKIN_BASEURL
							   valueFrom:
							   	configMapKeyRef:
							   		name: {{ .Values.global.configMapName }}
							   		key: SPRING_ZIPKIN_BASEURL
							  {{- end }}
							  {{- if .Values.config_enabled }}
							  - name: SPRING_CONFIG_IMPORT
							  valueFrom: 
							  	configMapKeyRef:
							  		name: {{ .Values.global.configMapName}}
							  		key: SPRING_CONFIG_IMPORT
							  {{- end }}
							  {{- if .Values.eureka-enabled }}
							  - name: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
							    valueFrom:
							    	configMapKeyRef:
							    		name: {{ .Values.global.configMapName }}
							    		key: EUREKA_CLIENT_SERVICEURL_DEFAULTZONE
							  {{- end }}
							  


{{- end -}}
```

i ove vrednosti moramo da specificiramo negde

jer ovde radimo sa .Values onda imamo values.yml tako ce da zna

##### Config map

```
{{- define "common.configmap" -}}
apiVersion: v1
kind: ConfigMap
metadata: ConfigMap
	name: {{ .Values.global.configMapName }}
data:
	SPRINT_ZIPKIN_BASEURL: {{ .Values.global.zipkinBaseURL }}
	SPRING_PROFILES_ACTIVE: {{ .Values.global.activeProfile }}
	SPRING_CONFIG_IMPORT: {{ .Valkues.global.confgiServerURL }}
	EURAKE_CELINT_SERVICEURL_DEFAULTZONE: {{ .Values.global.eurekaServerURL}}
{{- end -}}
```

values.yml

vidimo da imamo nesto .global,u Helm-u postoje 2 vrete varijabli

1. Global -> svuda mozemo da pristupimo 
2. Local -> samo u chartu



MI kada koristimo helm chart mozemo da ga pokrecemo kao lib ili kao app,ako je kao lib onda mozemo da stavljamo da ta lib bude zavistnost neke druge,ono kao dependency

Kako ovo pravimo da bude lib,nas values.yml ce biti empty .Svaki mikroservice ce imate values.yml koji ce sam da popunjava,zato se zove ovaj chart(helm create name) name-common



Kako hocemo da za svaki mikroservis napravimo novi folder npr account-microservice i udjemo u cmd i napravimo chart novi

```
helm create accounts
```

Izbrisemo templates i values jer cemo mi to da definisemo ili ubacimo lib

Kako da ubacimo lib sada onu sto smo napravili:

1. idemo u chart.yml

2. Napravimo

   

   ```
   .
   .
   .
   dependencies:
   	- name: ime -> ime lib
   	  version: 0.1.0 -> verziju lib nadjemo tako sto odemo u chart.yml biblioteke
   	  repository: file://../../imelib -> jer je nas common-lib u nasem loklnom fajl sistemu 			moramo da kazemo gde je.To pocinje sa file:// pa putanja,ovo idemo nazad jer je u istom 		 folderu nazad,mozemo dati i http ako je negde na repo
   	  
   ```

3. I sada kako ovaj mikroservice chart nema u tempalte nista mi moramo da ubacimo `deployment.yml` i `service.yml`.Sada se postavlja pitanje sto to opet pravimo?Razlgo je sto mi imamo vec to u lib koju smo ukljucili i ti templates imaju imena mi samo u novnokreirani template referenciramo taj iz lib

   ```
   {{- template "common.deployment" . -}}
   ```

   Ovo stavimo u novi `deployment.yml` i kako imamo ukljuceno lib koja ima template za deployment sa ovim imenom mi samo import taj,ali svakako mora da postoji fajl ovaj fajl makar on samo referencirao tamo.

4. To uradimo i za service

5. Zadnji korak je da u values ubacimo,imali smo one varijable i njihova imena

   ```yml
   deploymentName: accounts-deployment
   deploymentLabel: accounts
   replicaCount: 1
   image:
   	repository: easybytes/accounts
   	tag: latest
   containerPort: 8080
   service:
   	type: LoadBalancer
   	port: 8080
   	targetPort: 8080
   config_enabled: true
   zipkin_enabled: true
   profile_enabled: true
   eureka_enabled: true
   ```

   Ovde nemamo sifre i to ,u values smestamo te tako info koje nam nisu previse znacajne ako ih neko uzme

6. Sada a config map imamo 

7. 

   ```
   helm dependency build
   ```

8. Ovo je samo za mikroservise,moramo da napravimo jos 2 chart-a za dev i prod,e tu smestamo podatke o config mapama

9. Dodamo dependency ovde za svaki mikroservis i common lib

10. Dodamo `configMap.yml`

11. ```
    {{- temp template "common.configmap" . -}}
    ```

12. i sada values ubacimo

    ```yml
    global:
    	configMapName: ...
    	zipkinBaseURL: ...
    	activeProfile: dev
    	configServerURL: ...
    	eurekaServerURL: ...
    ```

    



```yml
deploymentLabel: accounts
service:
	type: ClusterIP
	port: 8080
	targetPort: 8080
```

MI moramo da instaliramo Helm da bi radili sa njim(kao sto instaliramo npm)

```
helm create name
```

kreirace folder za rad,i strukturu,i ostale fajlove koji su tu

- Chart.yml
- values.yml
- charts/ ->zavisnost ka drugim
- templates/
- helmignore

templates/

- hpa.yml -> sluzi za autoscale

- ingres.yml -> sluzi da expose microservies,routing

- service.yml

- deployment.yml

  Izbrisemo default fajlove u templates i values i napisemo sami.

  ovde pisemo one fajlove za deployment , configMap, services

  ```
  
  ```

  

Kada sve to odradimo samo instaliramo helm na k8s klaster.

jer radimo dev instaliramo dev deo(lakse je da cmd otvorimo u folderu gde smo instalirali foldere za dev i prod zbog navigacije i tu da se login na cloud)

```
helm install dev-depoloyment helm_dev_location
```

Nekad posle instaliranje na k8s mi hocemo da upgrade ako nesto promenimo

Kada promenimo neki od mikroservisa(neki helm) onda mi samo build 

```
helm dependencies build
```

ovo build u specificnom folderu gde je dev ili prod

```
helm upgrade deploymentHELMname folderLocationDev
```

### Rollback

```
helm rollback helmdeploymentname revisionnumber
```

### History

```
helm history helmdeploymentname
```

### Delete

```
helm uninstall helmdeploymentname
```

