# Arhitecture

## Clean Arhitecture

Ovo je `separation of concerns` tako sto se podeli aplikacija u razlicite delove(different layers)

![image-20230423171411354](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423171411354.png)



Vidimo jasnu separaciju delova aplikacije.

Svaka boja je razliciti deo softvera.

##### Dependency rule

Dependencies pokazuju unutar softvera,tako da entites nema nikakav dependenci .On je Raw i jedinstven.

To znaci da elementi mogu znati samo unutar njih,ne i van.Npr imamo entities koje ne znaju nista sto je spolja od njih.Gateways znaju samo unutrasnji krug,ne bi videli nita spolja kao sto je ovaj db ui...

Ako pojednostavimo sve ovo,ispada ovako:

![image-20230423171834961](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423171834961.png)



Ova strelica prikazuje "depends on" znacenje.Nista iz donjeg sloja ne bi trebalo da zavisi od gornjeg

### Frameworks and drivers

- User Interface
- Database
- External Interfaces (eg: Native platform API)
- Web (eg: Network Request)
- Devices (eg: Printers and Scanners

To sve sadrzi i radi:

- `Presenters` (UI Logic, States)
- `Controllers` (Interface that holds methods needed by the application which is implemented by Web, Devices or External Interfaces)
- `Gateways` (Interface that holds every CRUD operation performed by the application, implemented by DB)

### Application Business Rules

Provajduje nam `Use Cases` aplikacije,takodje odredjuje kada ce se koji `Gateway/Controller` zvati za koji use case

Koristi se `Dependency Inversion`  i `Polymorphism` da se lako odvoje delovi aplikacije

Source kod dependencies mogu samo da pokazuju unutar(inward) domain layer-a

Ova arhitektura se jos zove i `plugin and ports` arhitektura.Jer svaki deo mora da se prikljuci nezavisno kao plugin

Nedostatak ovoga:

1. Moramo da trosimo vise vremena na pisanje koda
2. Dupli kod 



Koristimo `Use Cases` koji sadrze biznis logiku

Dosta je slicno kao DDD

Ovde imamo Entites i Use Cases a u DDD imamo Agregates i Domain Services

### Enterprise Business Rules

Ovo je sloj koji sadrzi core-businsess ,ovo se skoro nikad ne menja,to je srz.Promene drugih slojeva nece uticati na ovaj sloj





Primer

![image-20230423181410934](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181410934.png)

Ne bi islo ovako zbog dependency rula

![image-20230423181551577](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181551577.png)

vidim oda se krsi,to resavamo preko  `Dependency Inversion Principle` tako sto ubacimo interfejs izmedju 2 sloja

![image-20230423181634234](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181634234.png)

![image-20230423181637585](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181637585.png)

![image-20230423181646912](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423181646912.png)

### Primer Order Service

![image-20230423182609025](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423182609025.png)

Ovako bi poceli da raidmo,i slika nam izgleda ovako.Ovo nije dobro jer nam Buisness Logic zavisi od RDBMS-a ,to ne valja jer mi hocemo da budemo fleksibilni da bi mogli da je menjamo kada hocemo.Svaka promena Data Layer-a mora da se proprati menjanjem Domain Layer-a,to ne valja,jer imamo DIREKTNU zavisnost

![image-20230423182909145](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230423182909145.png)

Ovako smo to resili

Napravili smo interface koji ima expose neke operacije,i te operacije mogui samo da se pozivaju.Nas Data LAyer ce sam da implementuje te operacije a DomainLayer ce da sadrzi referencu nad njima

Sada vise nisu zavisni

Ovo je glavni cilj `Clean Arhitecture` da mi koristimo `Dependency Inversion` zbog zavisnosti,i da uradimo `Polimorfizam` kao i `Dependency Injection`

-----------------------------------

Fora je da napravimo projekat tj module koji je npr `Domain` i tu nemamo nikakve logike vec samo klase koje su nam DTO,ENTITIES...

### Bitni koncepti:

- Entities -> objekti
- Use Cases -> ovde sadrze implementaciju biznis logike
- Infrastructure -> API,Gateway,Repositories,Configurations



Fora je da mi koristimo Use Cases kao elemente biznis logike,svaki use case ce raditi nesto ,kao one Sistemske Operacije iz PS-a

Flow:

![image-20230424004755644](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230424004755644.png)

1. An external system performs a request (An HTTP request, a JMS message is available, etc.)
2. The Delivery creates various Model Entities from the request data
3. The Delivery calls an Use Case execute
4. The Use Case operates on Model Entities
5. The Use Case makes a request to read / write on the Repository
6. The Repository consumes some Model Entity from the Use Case request
7. The Repository interacts with the External Persistence (DB,Cache,etc)
8. The Repository creates Model Entities from the persisted data
9. The Use Case requests collaboration from a Gateway
10. The Gateway consumes the Model Entities provided by the Use Case request
11. The Gateway interacts with External Services (other API, microservices, put messages in queues, etc.)

Bukvalno ovo `Repositories` je neka impl,npr ProductRepository koji sadrzi neke operacije koje radimo nad bazom,nisu samo CRUD operacije ciste vec ima neke logike i error handling,a core biznis logika je u `Use Cases`

npr metoda u repository

```
return categoryRepository.findAll().stream().map(category -> categoryRepositoryConverter.mapToEntity(category))
                .collect(Collectors.toList());
```

Tako da bi repositoryimpl imala autowired dependencty na npr ProductRepository