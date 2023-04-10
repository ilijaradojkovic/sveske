# Microservice Distribuded Tracing

Ova dva tool-a idu zajedno,mi sa TRACE ID i SPAN ID mozemo da vidimo na Zipkin UI sta nije ok

## Sleuth

auto-configuration for distribuded tracing

mi samo dodamo ovaj dependancy i sve ce on sam da uradi(auto-configuration)

Radi tako sto dodaje `traceId` i `spanId` na log,ovo radi pomocu filtera

Pattern pisanja ovoga tracing-a

[<APP NAME>,<TRACE ID>,<SPAN ID>]

app name->ime app

trace id-> jedinstven broj koji predstavlja celu transakciju

span id->jedinstven broj koji predstavlja deo transakcije

![WhatsApp Image 2023-04-10 at 1.42.08 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-10 at 1.42.08 PM.jpeg)







## Zipkin

Open-source data-visualization tool

Agregira sve logove 

![WhatsApp Image 2023-04-10 at 2.18.32 PM](C:\Users\Ilija\Downloads\WhatsApp Image 2023-04-10 at 2.18.32 PM.jpeg)

Zipkin se sastoji iz ovih delova

1. <span style="color:red">Collector</span> -> Kada trace data stigne u collector daemon,validira se ,cuva se i indekira se za lookup
2. <span style="color:red">Storage</span>->Zipkin podrzava in-memory storage,MYSQL,CASSANDRA...
3.  <span style="color:red">Querty service (API)</span> -> Ovo je nacin da se ti podaci extract
4. <span style="color:red">Web UI</span>-> vizualni prikaz

Ovde mozemo da vrsimo async i sync komunikaciju gde se podaci sa mikroservisa salju u ovaj storage(collector)

Ovo je depricated u spring boot 3.0 ,koristi se `Micrometer Tracing`

1. Na sve mikroservise dodamo dependency

```
spring-cloud-starter-sleuth
```

2. Sledeci korak je da dodamo Logger i da logujemo stvari na kontrolerima,kada dodamo ovu lib on ce sam na logove da dodaje sve



3. MI moramo da run zipkin server na dokeru

```
docker run -d -p 9411:9411 openzipkin/zipkin
```

i sada mozemo da pristupimo UI stani `localhost:9411`

4. Dodamo dependency u spring-u za zipkin

```
spring-cloud-sleuth-zipkin
```

5. Konfiguracija u yml fajlu

   ```yml
   spring:
   	sleuth:
   		sample:
   			percentage: 1 -> koliko % log fajla ce sleuth slati zipknu da pravi statistiku ovo 								je 100%,po defaultu salje 10%
   	zipkin:
   		baseUrl: http://localhost:9411
   			
   ```

   

Ovo za sada ce da push sve sync to je malo glupo jer blokira app,hocemo to da ucinimo async

npr koristicemo kafku za asinhrono slanje

1) Pokrenuti docker container za kafku

2) Ubaciti kafka dependency u svim mikroservisima(svaki ce da salje async)

3) ```yml
   spring:
     zipkin:
       sender:
         type: kafka
   ```

   