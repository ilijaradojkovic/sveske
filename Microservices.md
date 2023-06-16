# Microservices

Pre je bio popularana arhitektura aplikacija **monolith**.Monolith je arhitekturni naziv za aplikacije koje imaju sve strpano u jedan projekat.

![image-20230408190958794](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408190958794.png)

U monolitu se nalazi sve u jednom projektu,i sve je usko povezano(tightly coupled) sto znaci da promena na jednom delu jako utice na ostale delove,isto tako je tesko zameniti jedan deo ovoga.Na primer sutra hocemo da izmenimo payment ali to ce prouzrokovati promene svuda.Takodje imamo 1 bazu.

Ovaj nacin je zastupljen i dan danas,treba pazljivo birati arhitekturu koja ce da zadovolji potrebe.Nekada je ovo bolja arhitektura ,ali nekada nije.

![image-20230408191014656](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191014656.png)

Ovako bi izgledala sira slika mikroservisa.Svaki mikroservice je zaseban projekat koji pravimo.Svaki mikroservice je zasebna jedinica koja zna da radi samostalno,ona ima svoju bazu sa kojom komunicira ii ma samo one tabele koje su joj neophodne.Tako person microservice nece imati tabele koje su vezane a order jer to nije posao person microservica.Ovde imamo jasnu podelu **separation of concerns**,i **single responsibility**.

Ako zumiramo jos vise da vidimo o cemu je tu rec dobijamo ovu sliku:

![image-20230408191035407](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191035407.png)

Ovde kada zumiramo vidimo sta sve jedan mikroservice ima,vidi se da je zasebna aplikacija kako sadrzi **controller** koji prima neke zahteve ,**service** koji ih obradjuje i **repository** koji te zahteve prevodi u jezik baze

Mi sutra mozemo da potpuno zamenimo jedan mikroservice drugim samo sa istim endpoints i sve ce da radi kako treba.

Razvijanje microservice app je u pocetku teze,ali za velike staze je mnogo fleksibilinije i lakse,dok je monolit laksi za razvoj na pocetku ,a kasnije dosta tezi za odrzavanje.

# Microservice Patterns

Postoje vise patterna 

## Database Patterns

Postoje paterni za dodelu db mikroservisima,postoje 2.

### Multiple DB (1 for each microservice)

Ovaj pattern je najzastupljeniji i odnosi se na to da svaki mikroservice sadrzi bazi za sebe.U svakoj bazi ce se nalaziti samo one tabele koje su neophodne za taj mikroservice.Tako na primer u mikroservisu Payment necemo imati tabelu users sa email i password poljima jer to nije neophodno i jednostavno je lose .Svaki nas projekat bi imao konekciju sa bazom u dependency-ju.Tako cemo imati mogucnost da svaka baza bude razlicita,jedna moze biti mongo,druga mysql itd.

![image-20230408191049849](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191049849.png)

### One DB(1 fro all microservices)

Ovo se smatra antipaternom,lakse je za upravljanje transakcijama jer je sve u istoj bazi,ali je neodgovarajuci.

![image-20230408191104219](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191104219.png)

## Paterni izdvajanja mikroservisa

Kako da izdvojimo mikroservise ,i da znamo sa kojima radimo?

Postoje neke stratigije pristupa.

Ako se jedana karakteristika mikroservisa nalazi na 1 mikroservisu ,a baza bi imalo smisla da se nalazi na drugom,kada imamo takvu situaciju da je logika pripadajuca jednom ,a baza drugom onda se izdvaja nov mikroservis.

### Na osnovu domena

Prepoznamo domenske objekte i ujedinimo ih ako je potrebno redi jednostavnosti.Pre je bio trend da se mnikroservisi izdvajaju na sitne celine,danas je da se izdvajaju na vece objekte tj da te manje mikroservise ujedinimo u vece.

 

 

## Komunikacija mikroservisa

Kako da vrsimo komunikaciju izmedju mikroservisa,one strelice sto idu iz jednog u drugi.

### REST Api

Jedan nacin je da koristimo rest api.Svaki mikroservise tj njegova komponenta controller ce da sadrzi endpoint kojima cemo pristupati

![image-20230408191119169](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191119169.png)

### Messaging       

Ovo je nacin komunikacije preko kanala.Tipican predstavnik je **kafka.** Obicno su ovi kanali asinhroni,poruka se ostavi u kanal i onaj ko osluskuje cita iz kanala

![image-20230408191222946](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191222946.png)



## Upravljanje mikroservisima

### Multiple Instances

Svrhaovoga je da cemo mi pokrenuti neke mikroservise

![image-20230408191244291](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191244291.png)

I oni ce ovako raditi  

Medjutim sta se desi ako jedan od njih prestane da radi?Kako cemo to handle

![image-20230408191257528](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191257528.png)

Ako jedan mikroservice padne mi cemo morati rucno da ga pokrecemo,sto nije dobro.Postoje tools koji to rade umesto nas.To je **Kubernetes** zajedno sa **Docker**-om 

Oni nam omogucavaju da pokrenemo vise instance jednog istog servica i upravljaju padovima mikroservisa,ako jedan padne on ce automatski pokrenuti drugi.

![image-20230408191307402](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191307402.png)

### Service Discovery

Sada imamo vise njih koji ce se run.Dolazimo do novog problema ,a to je kako mi znamo koji je ip novog koji je prethodno ugasen.Prosta situacija mi smo pokrenuli mikroserice na ip 192.168.1.103 i znamo da ako hocemo da gadjamo njega samo ukucamo taj ip.Ali ako se on ugasi i ponovo pokrene njemu ce se ip automatski dinamcino dodati kako cemo znati koji je sada ip?Ovo je sve posao kubernetes-a koji obavlja za nas,taj proces se zove **Service Discovery**

Service Discovery je princip knjige sa brojevima gde znamo gde je nas server,svakoj grupi mikroservisa se dodaje ime pa ce unutrasnji dns da preusmerava zahtev na odgovarajuci ip.Kada se mikoroservise pokrene on se bind za to ime tu grupu,tako kubernetes zna da je taj mikroservise pokrenut i da se nalazi u grupi.

Recimo situaciju da imamo Person grupu na kubernetes-u i svaki put kada se pokrene person mikroservice on pozove metodu da ga kubernetes doda u tu grupu.Kada je u toj grupi samo preko imena te grupe Person ce dns odraditi posao i preusmeriti zahtev ka toj ip addr.

Servie discoveri je kao velici recnik podataka tj podataka mikroservise,tj podataka o ip-ju i portu.Interna komunikacija izmedju mikroservisa

### Load Balancing

Kada imamo ovako vise mikroserisa i znamo da se vise pokrecu i gde se nalaze(koj ip) ,svrha postojanja vise instance mikroservisa je da bi svaki mogao da radi posao.Zamislimo situaciju gde imamo 5 klijenta,dovoljan ce biti 1 mikroservis da obradi zahteve,ali kako se oni povecavavaju na tipa 5000 moracemo da pokrenemo vise instance mikroservise kako ne bi kocio(npr kada je black Friday).Pokretanje vise intsanci mikroservise radi kuberentes za nas automatski.Problem koji nastaje iz ovoga je kako ce korisnik da zna kom mikroservisu da salje zahtev?Ovo nije problem koji resave Service Discovery jer je on za internu komunikaciju izmedju mikroservisa.Nama je potrebno da znamo da klijent gadja drugu intsancu mikroservisa,to radi load balancing.On balansira dolazece zahteve u mirkoservis na razlicite instance mikroservica,kako svih 5000 zahteva ne idu u 1 on ce pomocu **Round Robin** algoritma da podeli broj zahteva sa mikroservisima i podjednako da posalje svima zahteve.To nam Kubernetes omogucava automatski.

 

 

## Deploying microservices

Kada napravimo sve mikroservise i pokrenemo ih sa play dugmetom to je ok na nasem lokalnom nivou.Kako cemo to prebaciti u produkciju?

Potrebno je to da ode na internet da svima bude dostupno.To se radi preko cloud-a.Potreno je zakupiti proctor na cloud-u i deploy svaki mikroservice.

Svaki mikroservice ce biti 1 image container u docker-u,i svima njima ce da upravlja(orchestrator) kubernetes.

![image-20230408191317949](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191317949.png)

Api gateway je centralizovana ulazna tacka koja zahteve obradjuje i salje ostalim mikroservisima

Tako da cemo mi gadjati api gateway koji je poseban projekat tj app i ona preusmerava requestove.Njen api je izlozen public networku.

Single entry point

Data transformation(ako je potrebno dolazece podatke transformirasti pre slanja drugom servisu)

Za ovo postoje gotova resenja Zuul,Netty,Finagle

![image-20230408191327002](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191327002.png)

Vidimo da ako nemamo api gateway on ce direktrno da gadja ove mikroservise,pa ako nam trebaju vise informacija sa razlicitih mikroservisa kao ovde da nam treba sa person microservice i order microservice mi cemo imati 2 poziva,ali ako nam treba n informacija sa n razilictih mikroservisa imacemo n poziva sto niej dobro.Zato nam api gateway sluzi da on 1 pozim preusmeri ka ostalima i te podatke ujedini i nama posalje 1 odgovor

Takodje cemo na api gateway-ju da uradimo validaciju objekata koji se salju od user-a

## Microservice Security

Pitanje je gde cemo vrsiti proveru identiteta i uloga requesta koji se salje?

Da li cemo to uraditi na svakom mikroservisu?

Da li cemo to uraditi samo na jednom mikroservisu?

Raditi ovo na svakom mikroservisu nema poente ,posebno zbog api gateway-ja kroz koje prolaze svi zahtevi.Kada zahtev stigne u api gateway:

1) Api gateway validira i salje dalje
2) Prosledi validaciju auth serviru i ako je ok onda nastavi dalje

![image-20230408191347315](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191347315.png)

![image-20230408191354965](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191354965.png)

Opste poznati auth servr-i **Okta**,**Shiro**,**Keycloak**.Ta resenja mozemo da integrisemo u nas sistem da ne pravimo nas.

Sada se postavlja pitanje ako se mi uspesno auth i zahtev ode na order microservice,i order treba da pozove payment microservice kako ce payment znati da smo mi uspesno auth.Kako da komuniciramo identitet izmedju mikroserisa?

![image-20230408191408455](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191408455.png)

Odgovor je preko **Access Token**.Svaki token sadrzi informacije,i svaki service ce da validira sam token,to je ono sto saljemo service da bi decode info,da bi video da li moze da ordari neku akciju

Fora je da cemo mi postaviti gateway koji je `client` (registrovan client na auth server-u) i savki request koji dodje kod njega i nema odgovarajuci jwt token ce otici na loing stranu `autorizacionog servera` 

##  Puna Slika:

![image-20230408191427285](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191427285.png)

## CQRS

![image-20230408191442976](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230408191442976.png)

Mi saljemo u command service da se uradi crud operacija a command service ce da to prosledi na kafku,gde ce query service da slusa i da ubaci u bazu.Jer baza mora da bude na 1 mikroservisu kada su ovako odvojeni.Kornisk ce da salje command service crud operacije ali u pozadini on ice da se prosledjuju na query service jer je tu baza,a kada mu treba nesto on ce da gadja query service

## Komunikacija mikroservisa

Izmedju samih mikroservisa unutra treba biti kafka,tj neki broker.Izmedju api gateway i mikroservise mora biti rest

# Events in microservices

Events posotje kao 2 tipa:

1)Notification

2)Data replication

Notification eventi su oni koji obavljaju core biznis logiku,kada se nesto desi salje se event drugim mikroservisima da nastave sa radom bizins logike,npr za kupovinu 

Data replication je to sto ostali mikroservisi sadrze kopije tabela iz drugik mikroservisa,pa promenom originalne tabele salje se event da se replicira ta tabela tj da se desi update kod ostalih mikroservisa

## Resiliance

U nasem mikroservisnom okruzenju,`resiliance` se odnosi na nacin na koji sistem ima sposobnost da se oporavi od gresaka i failures.Resiliance sistem je onaj koji moze da nastavi sa radom kada se neka greska desi.

Ovo mozemo resiti primenom nekih paterna:

1. Circuit breakers
2. Bulkheads
3. Timeouts
4. Retries
5. Fallbacks

### Circuit breaker pattern

Ovaj patern radi tako sto nadgleda `health` samog servisa,i ako servis postane unavailable ili unresponsive, on neomogucuje da se ostali requests posalju.

Kako on radi?

1. Client posalje zahten an remote service kroz circuit breaker.
2. Circuit breaker prati response time i error rate of service over time
3. Ako service postane nedostupan il ivrati error,circuit breaker otvara `circuit`
4. Dok je circuit(kolo) otvoreno circuit breaker ce odbiti bilo koj pokusaj da se posalje request ka tom servisu opet
5. Ako servis odgovori uspesno,cirucit breaker ce se zatvoriti i radimo normalno

Bitno je da razumemo da je ovo ideja `kola` sve dok je kolo otovoreno necemo mocu da saljemo na taj servis opet

### Rate Limiting(Client focus)

Ogranicavamo pozive na api po klijntu.Kazemo da mozemo na endpoint GET /product da imamo 100 poziva per second npr.

Koristimo 429 kod

### Load Shedding(System focus)

Ogranicava pozive u odnosu na stanje sistema.Mi smo u rate limitingu imal ida zavisimo i ogranicavamo od stanja klijenta tj koliko pozia je imao,a ovde mi ogranicavamo u zavisnosti od backend stanja,CPU ...

Koristimo 503 kod



