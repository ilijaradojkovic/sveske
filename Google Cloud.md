## Google Cloud

`Cloud SDK` ovo skinemo da bi mogli da izvrsavamo cmd komande nad cloud-om.

Fora je da mi pokrenemo k8s cluster i da se preko cmd-a povezemo na njega,kada se tu povezemo mozemo da preko UI-a ubacujemo container-e i podove,a mozemo i da se preko cmd koriscenjem kubectl-a

Obicno cemo da imamo yml fajl za deployment i service pa cemo njih da ubacujemo u nas cloud cluster

Buklno radimo isto sto i lokalno samo sto je klaster na cloudu i mi se konektujemo na taj klaster

![WhatsApp Image 2023-04-12 at 1.14.19 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 1.14.19 PM.jpeg)

Fora je sto mi kada izbacimo miksroservise kao podove oni ce imati svoju ip adresu,i svi ce moci da im pristupe.

Ovo nije idealno,samo bi trebalo da kroz `spring cloud gateway` dolaze zahtevi

Necemo da expose sve ostale vec samo gateway

Postoje 3 servisa koje se koriste na k8s na cloud-u:

1. ClusterIp Service -> ovo je default service koji koristi internal Cluster IP da expose pods.U Cluster IP -u servisi nisu dostupni za external access,samo za internal je moguce komunicirati.Samo u cluster-u medjusobno mogu.Kao da ima firewall 
2. NodePort Service -> on expose za svet spolja ,pristupa se pomocu <NodeIP>:<NodePort>
3. LoadBalancer Service -> Ovo je kao NodePort samosto se kreira loadbalancer ,nodeport se menja ali loadbalancer service je statican

### Cluster Ip Service

Niko ne moze da pristupi klasteru van,

![WhatsApp Image 2023-04-12 at 2.01.54 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 2.01.54 PM.jpeg)

Namestimo da ovaj tip bude,default ce biti svakako ovo

![WhatsApp Image 2023-04-12 at 2.02.47 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 2.02.47 PM.jpeg)

Target port je onaj port kome mi pristupamo mikroservisima,a port je port za cluster ip

Ovo je cluster ip za deployment account,on njime upravlja,tj kao da ih grupise,kao neki gateway.

Za cluster ip njemu treba port 8080 da bi pristupio account-u,a mi gadjamo clouster ip port to je 80,ovaj cluster ip se takodje ponasa kao lb

Kako cemo resiti onaj problem gore?

jer radimo sa helm-om ,ili ako sami pisemo yml fajlove,samo stavimo da 

```
service:
	type: ClusterIP
```



na one mikroservise za koje necemo da expose u spoljni svet,dok ce na gateway koji zelmo da bude expose staviti 

```
service:
	type: LoadBalancer
```

### NodePort Service

Svaka instanca poda se nece gupisati kao ako je account pa cu sve accounts da grupisem pod jednim ip-jem kao sto je u service ip,vec svaka instanca ce biti jedna grupa 

pored target i port imamo i `nodeport`

![WhatsApp Image 2023-04-12 at 2.19.58 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 2.19.58 PM.jpeg)

![WhatsApp Image 2023-04-12 at 2.20.41 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 2.20.41 PM.jpeg)

Ovo nije pozeljno da se radi,najbolje je service ip

Mi bi ovde pristupali pomocu nodeporta,napredva verzija nodeport-a je `loadbalancer`

## LoadBalancer

Vidimo da on kombinuje NodePort tako sto ubaci loadbalancer pa ne moramo da brinemo za ipnodova

![WhatsApp Image 2023-04-12 at 2.37.34 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-12 at 2.37.34 PM.jpeg)

Service koji hocemo da bude expose spolja -> Load Balancer service

Service koji hocemo da ne bude expose spolja -> Cluster ip service 



## OAuth2 

Do sada sto smo uradili stavili smo da ovi budu private tj da nisu expose mikroservisi,samo je gateway expose

Spring cloud gateway prima request od svih 