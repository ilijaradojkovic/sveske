# Monitoring Tools

- **Actuator** -> Ovo smo vec radili,ovo je veoma basic.Problem se javlja kada imamo vise instanci mikroservisa i svaka instanca ce imati port pa ce actuator za 100 mikroservisa moramo da posetimo 100 endpointa za healt

- **Prometheus** -> Ovo sakuplja podatke sa vise mikroservisa radi statistiku i imamo lep UI,ovo radi tako sto pull data,mi ne saljemo njemu podatke,vec pomocu actuator-a mi expose endpoints a prometheus sakuplja te endpoints tako da on samo pull .Ovde se javlja problem kod formata podataka,jer ako format nije dobar tj nije onaj koji prometheus ocekuje imacemo problem,zato koristimo `Microemter`
- **Grafana** -> Ova skuplja podatke iz Prometheusa i daje nam dobar ui,notifikacije nam daje,grafove
- **Micrometer** -> Micrometer ce pripremiti taj format podataka da bi prometheus razumeo

## Micrometer

Dodamo dependencies

```xml
<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-core</artifactId>
</dependency>

<dependency>
	<groupId>io.micrometer</groupId>
	<artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Da bi mogli da pisemo nase custom metrict moramo da dodamo spring aop kako bi nam bilo lakse

```xml
<dependency>
	<groupId>org.springfremework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Ako hocemo da imamo uvid u vreme mi cemo napraviti brean tipa

```java
@Bean
public TimedAspect timedAspect(MeterRegistry registry){
	return new TimedAspect(registry);
}
```

i sada mozemo da ubacimo anotaciju za merenje vremena requesta

```java
@Timed(value="getAccountDetails.time",description="Time taken to return Account Details")
@PostMapping
public AccountDetails getAccountDetails(){
	...
}
```

## Prometheus

Mi kada radimo sa prometheus-som moramo da napravimo `prometheus.yml` fajl gde ce se nalaziti prometheus config stvari



```yml
global:
	scrape_interval: 5s -> uzima podatke na svakih 5s
	evaluation_interval: 5s -> evaluira podatke na svakih 5s
scrape_configs:
  - job_name: 'spring-boot-app' -> application name
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080'] -> kako da pristupi nasoj app,jer on pull data treba mu ip kao i 									port
    
```

Ako bi imali vise mikroservica svaki ovaj job_name bi bio ime jednog mikroservisa,jer je ovo fajl koji je na istom nivou kao docker-compose sto je root nivo.

Prometheus cemo objaviti na docker-compose:

```yml
prometheus:
	image: prom/prometheus:latest
	ports:
		- "9090:9090"
	volumes:
		- ./prometheus.yml:/etc/prometheus/prometheus.yml ->kopiraj ovaj fajl u  etc...
	networks:
		- mynetwork
```

i mozemo da idemo localhost:9090 jer je to prometheus lokacija da vidimo UI

## Grafana

Samo dodamo grafanu 

```yml
grafana:
	image: "grafana/grafana:latest"
	ports:
		- "3000:3000"
	environment:
		- GF_SECURITY_ADMIN_USER=admin
		- GR_SECURITY_ADMIN_PASSWORD=password
	networks:
		- mynetwork
	depends_on:
		- prometheus
```

Kada odmemo na localhost:3000 na ui Grafane moramo da dodamo prometheus da bi ona skuoljala info

Moramo da idemo na grafana dashboars da skinemo i onda da import u grafanu