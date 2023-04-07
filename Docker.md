Docker je tool koji run containers.

Moramo znati razliku izmedju VM i containera.

# VM

Virtual machine je podsistem koji dize novi operativni sistem i pokrece aplikacije nad njim.

![image-20230331232309564](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232309564.png)

Vidimo da pokretanjem VM za neke programe se pokrece ceo novi os n avec postojacem os-u.Ovo je heavy operacija i za pokretanje pojedinacnih mikroservise nama os nije potreban.Ovo ce zauzeti dosta memorije.

# Container

![image-20230331232336405](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232336405.png)

Vidimo da container ima mnogo manje komponenti koji su potrebne da se run app.On je vise lightweight pa zato i koristimo,ovo runtime engine je za docker docker hub koji pokrecemo i stavljamo pojedinacne containere da se run

# Docker Components

Postoje nekoliko glavnih koncepata koji objasnjavaju docker

1)Image

2)Container

3)Network

4)Volume

 

## Image

Ovo je blueprint kako pravimo,da bi se container napravio mora postojati image pomocu kojeg ce se container napraviti,container se pravi nalik na image.

<span style="color:#ff4d4d">Container = Image + env variables</span>

![image-20230331232402106](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232402106.png)

Mi kada run image napravimo container u stvari.Neki container-i ce traziti env variables da bi se pokrenuli,kao sto je default user,port…

Mi mozemo da od iste image napravimo vise containera samo podesavamo config tj env variables.

Image skidamo sa docker hub-a,postoje vec gotove images npr za mysql image se zove mysql i samo je pull

docker pull imagename

on ce automatski downalod sliku i ubaciti je kod nas.

Tako da mi mozemo da napravimo od naseg mikroservise image i samo da je pokrecemo na razlicitim containerima,i razlicitim env variables vise puta.

## Container

Ovo je runnable instance image-a.Kada pokrenemo image stvara se container

**Komande:**

<span style="color:#ff4d4d">docker ps</span> -> vidi sve running container-e

<span style="color:#ff4d4d">docker run [options] image</span> -> pokrece image kao container(tj run image)

​        **--name** **ime** -> dodaje ime container-u

​        **--net** **ime** -> dodaje container na network

​        **-p** external:internal -> dodaje port container-u

​        **-e** **ime=val** -> set environment variable

​        **--ip=ip..** -> postavlja container na ip

​        **-d** -> detached mode,run container in background

​        **-it** -> run container interactively(exe cmd while container is running)

​        **--rm** -> kada stop container on se izbrise

​        **-v path** kreira volume za container(path->ime:destination(myval:/app))

<span style="color:#ff4d4d">docker inspect imecontainera</span>-> ovo nam daje uvid u sve info container-a

**
**

 

## Volume

Ovo je fajl u kome se pisu nasi podaci sa kojima radi container.

Npr imamo db mysql,kada radimo neke upite nad njom desavaju se promene u bazi,gde se te promene cuvaju u container-u?

Odgovor je u volume,to je fajl u kome ce nasa mysql baza ili neki container pisati promene,taj fajl mozemo da share izmedju vise container-a .Ako nemamo volume i u nasu bazu smo pisali stvari posle brisanja tog container-a ce se obrisati is vi nasi podaci jer ih nismo store na volume,ako ih store na volume ne samo da cemo mocu da ih share izmedju container-a,vec cemo ako se container iskljuci moci da imamo konzistente podatke.

**Komande:**

<span style="color:#ff4d4d">docker volume create ime</span>

<span style="color:#ff4d4d">docker volume ls</span>

<span style="color:#ff4d4d">docker volume inspect ime</span>

<span style="color:#ff4d4d">docker volume rm</span>

 



 

## Network

Po default svi kreiraju svoj zaseban network pa je komunikacija teza jer su svi izolovani.

Default network type za sve je bridge type.

Mi tome pristupamo pomocu exposed port-a.

Postoje nekoliko vrste tipova mreza:

1)Bridge

2)Host

3)MACVLAN

### Bridge type

Po defaultu ce napraviti bridge type uvek.Svaki put kada napravimo container on napravi za njega bridge network,tako da ako napravimo 2 container-a imacemo 2 networka ,2 bridga i oni medjusobno nece moci da komuniciraju

```cmd
docker network create --driver bridge
```

Bridge je internal docker network,bukvalno cemo imati switrch unutar docker-a i na jnemu ce biti povezani bridge networks

Mi imamo DNS u bridg-u pa mozemo da komuniciramo tj da dodjemo do ip-ja ili preko inspect-a container-a ili preko imena samog container-a jer radi DNS

![image-20230407230555699](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407230555699.png)



Svi se nalaze na istoj mrezi u bridge.Moramo da expose port

Host  type

![image-20230331232536791](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232536791.png)

Ovde nemamo izolaciju network avec container postavlja ravnopravno sa host racunarom.Kako je ravnopravan sa host racunarom ne moramo da expose port.

 

### MACVLAN type

Container-e cini neazvisnim i omoguicava da se povezu ravnopravno kao mypc na local router.

​    

Nezavisne masine postaju container-i.Bukvalno se ovi container-i konektuju na nas ruter i dobijaju ip adrese



![image-20230407230723610](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407230723610.png)

Ovde moze dodji do problema jer neki swicevi nece moci da handluiju da imamo vise mac adresa,svaki nas container ce imati za sebe mac adresu

Ova slika gore nam pokazuje kako bitrebalo da bude,ali u stavari izgelda ovako

![image-20230407230846796](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230407230846796.png)

Vidimo da su svi container-i povezani na 1 ulaz u mrezu?Tako je i to stvara problem pri dodeljivanju mac adrese,to se zove `promiscnons` mi to moramo da omogucimo na ruteru

![image-20230331232633279](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232633279.png)

**Komande:**

<span style="color:#ff4d4d">docker network create ime</span> -> po default-u pravi bridge type

<span style="color:#ff4d4d">docker network ls -></span> prikazuje sve networke

<span style="color:#ff4d4d">docker network create –d macvlan –subnet subnet –gateway ip –o parent=ehpos3</span>(interface of machine host)

## MACVLAN  

## IPVLAN

## IPVLAN L3



## None







 

# Docker compose

Ovo je nas file koji rucno pisemo,i nacin da pokrenemo vise container-a odjednom koji su medjusobno zavisni.

Npr Kafka-Zookeeper

1)Napravimo **docker-compose.yml** fajl

U fajlu imamo 5 glavna elementa:

- <span style="color:#ff4d4d">services</span>
- <span style="color:#ff4d4d">volumes</span>
- <span style="color:#ff4d4d">configs</span>
- <span style="color:#ff4d4d">secrets</span>

 Ovo su glavni elementi i oni imaju decu:

```
Service
	ImeService
		build
		image: imeImage -> pravi na osnovu ove slike
		container_name: name -> ime container-a
		deploy: ->
			:
		ports:
			-"internal:external"
		depends_od: -> onaj servis zavisi od ovih,pa ce se run posle ova dva
			-imeService1
			-imeService2
		environemnt:
			ENV_VARIALBE: VALUE
		networks:
			- networkName
		
```

Service:

​        **container_name:**

​        **image:**

​        **ports:**

​                **-“external:internal”**

​        **networks:**

​                **-ime**

​        **configs:**

​                **-config**

​        **secrets:**

​                **-secret**

​        **depends-on:**

​        **environment:**

 

network:

​        **driver:type**

 

 

 

Komande

**docker compose up** -> sluzi da pokrene docker file,moramo se nalaziti u istom prostoru gde je docker compose file,jer ce        on trazivi ovom komandom u proctor gde je cmd trenutno

# Docker File

Ako hocemo da deploy nasu app u container potreban nam je Dockerfile,ovako se mora nazvati.

To je file koji sadrzi instrukcije kako da izvrsi container app.Koristimo docker file da build container.

 

<span style="color:#ff4d4d">FROM img</span> -> na osnovu ove img pravi container

<span style="color:#ff4d4d">EXPOSE internal:external </span>->na koji port da pristupimo

<span style="color:#ff4d4d">ADD /target/nasjar nasjar </span>-> gde se nalazi nas exe file koji pokrece nasu app

<span style="color:#ff4d4d">CMD command </span>-> izvrsava cmd komandu

<span style="color:#ff4d4d">ENTRYPOINT [“cmd”]</span>   **->**ovo je specific exe,ulazna tacna za izvrsenje containera,ovde bi bilo nesto za pokretanje jar fajla jer nam je add jar file java –jar imejara

<span style="color:#ff4d4d">ENV var=vrednost </span>

<span style="color:#ff4d4d">COPY from to</span>

<span style="color:#ff4d4d">VOLUME path</span>

npr: Imamo app koju zelimo da ubacimo kao container

1)Napravimo dockerfile

2)Podesimo dockerfile

3)Izvrsimo **docker build . –t ime** (Ova . je dockerfile location,to nam je trenutni direktorijum .)

Ako se nalazimo u direktorijumu gde i dockerfile oznacimo sa , trenutnu lokaciju,ne moramo da specijalizujemo  lokaciju dockerfile jer smo mi u istom folderu kao i on

On ce napraviti image

4)Podesiti image i run

![image-20230331232734445](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230331232734445.png)

 Vidimo da app ima zavisnosti tj on koristi mysql i aws mi da bi pokrenuli container mi moramo da namestimo ove dependencies da bi se uopste app pokrenula,u proslom primeru nismo imali zavisnosti

U properties file (Java spring)

Spring.datasource.url=jdbc:mysql**://${MYSQLHOST:localhost}**

Ovo MYSQLHOST je env variable koju ako ne specificiramo uzece localhost,ali ako specificiramo tu varijablu prilikom pokretanja app on nece uzeti localhost.

Fora je da cmd kada run docker container da prosledimo -e ime-vrednost jer je to -e environment



 Napravio sam app

```
spring:
  datasource:
    username: postgres
    password: admin
    url: jdbc:postgresql://localhost:5432/test
    driver-class-name: org.postgresql.Driver
```

i instalirao sam postgres na dokeru

1)Onda sam ovu app pokrenuo i ona se povezala na docker container `postgres` jer postoji port forwarding i sve je ok.

2)Onda sam dodao ovu app da bude container i ostavio ovo u konfiguraciji da bude localhost,al ito nece da radi jer se onda taj nas app container nalazi u drugom docker networku i ne moze da prepozna localhost,vec moram ip tog container-a.

da bih video ip container-a moram:

```
docker inspect containerName
```

i dobio sam 172.17.0.2 sada umesto localhosta sam zamenio sa ovim i ono radi jer zna gde je container.Ili mogu ako su u istom networku da dodam samo ime tog container-a i kako su u istom networku(bridge) on radi DNS i nadje ga po imanu odmah

1. Bridge network: By default, Docker creates a bridge network for each Docker host. Containers connected to the same bridge network can communicate with each other using their IP addresses. Docker assigns each container a unique IP address on the bridge network, and also assigns a DNS name to the container, allowing other containers to communicate with it using the DNS name.

   Ovo je to 

   

   ```
   docker exec -it gateway sh
   ```

   

Ovime mi udjemo u sam container u njegovu konzolu,to je isto kao kada smo isli u docker desktop i ono console 

Ovde mozemo da ping ostale da vidimo da li ovaj ima pristup njima tj da li ih vidis

MI u bridge network ako ne expose port necemo moci da mu pristupimo

When you run a Docker container without specifying a network, Docker will create a new network for the container to use. By default, this network is a bridge network, which provides a private network space for the container and allows it to communicate with other containers and the host machine.

If you run a container without a network and do not explicitly expose any ports using the `-p` option, the container will only be accessible from within the host machine, and other containers will not be able to communicate with it directly.

However, you can still connect to the container from other containers or the host machine using the container's IP address or hostname. You can find the container's IP address using the `docker inspect` command, or you can use the container's hostname if it has one. The hostname is usually the container's name, which can be set using the `--name` option when running the container.

Mi u ovom brdige imamo DNS pa mozemo u samom bridge type(nas user defined npr testing ali je driver type bridge) da preko imena dodjemo do ip



Ovo je kada imamo depends na config server pa moramo da sacekamo malo da se load config server pa tek onda mikroservis da ga prepozna da radi

```
deploy:
	restart_policy:
		condition: on-failure
		delay: 5s
		max_attempts: 3
		windows: 120s
```

Mi u docker compose  moramo da kazemo image: pa koja je

ako ta slika nije sa docker huba tj nije vec definisana npr:

```yml
kafka:
	image: kafka:2.3.0 -> ova slika je sa docker huba i on ce moci da je pull


account:
	image: account -> ova slika je sa naseg lokalnog docker-a,tj mi smo pre pokretanja docker-compose up pokrenuli docker build na nas Dockerfile u account-u i time izbildali sliku pa je ovde mozemo navesti jer ona postoji
	
	
reservation:
	image: reservation -> ovde nismo build kao u prethodnom koraku i docker ne zna gde je slika ,mi moramo sami da mu kazemo da izbilduje automatki ako je nema,to raidmo preko:
	
	
services:
  web:
    build:
      context: ./webapp  ->Gde se nalazi
      dockerfile: Dockerfile ->Koj fajl da gleda
      
services:
	web:
		build: /Reservation -> a mozemo samo i ovako da mu damo putanju
```

