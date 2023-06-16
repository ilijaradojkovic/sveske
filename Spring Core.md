# Spring Core



Kljucne reci:

1. <span style="color:red">Bean</span>
2. <span style="color:red">IOC</span>
3. <span style="color:red">Bean Facory</span>
4. <span style="color:red">Application Context</span>



U Spring aplikaciji nasi spring objekti ce ziveti u spring container-u.

![image-20221220222237689](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221220222237689.png)

Spring ce kreirati objekte povezati ih zajedno(wire them),konfigurisati i upravljati njihovium lifecycle

Spring Inversion Of Controll je DI mehanizam.

<span style="color:red">Bean</span> je objekat kojim upravlja Spring IOC ,to je instancka klase ,kad kazem upravlja znaci da upravlja lifecycle ,konfiguracijom, gde ce se ubaciti

Ovo je princip koji spring implementir.Proces gde objekti definisu dependencies tj objekte sa kojima rade.Kada se kreira bean onda se sve dependencies ubacuju u IOC.

- org.springframework.beans
- org.springframework.context

Ovo su glavni paketi za ovaj rad

<span style="color:red">Bean Factories</span>  je glavna za stvaranje bean-ova.To je najjednostavniji container u spring-u koji pruza podrsku za DI.

<span style="color:red">Application Context</span> je implementacija ovoga koja pruza isti nacin rada,samo sa modifikacijama i uglavnom se najvise koriti.

1. Omogucava I18N
2. Provajduje nacin da genericno ucitava fajlove
3. Moze da publish events

Jedna od razlika je nacin ucitavanja beanova,i to static beanova.Bean Factory ce ucitati bean samo po pozivu getBean metode tj ucitace lazy,dok ce Appliation context raditi isto ali static beanove ucitati odmah kako ne bi cekao da se oni prave u runtime-u.

Application Context je implementacija IoC container-a

![image-20221220224919753](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221220224919753.png)

Nacin na koji IOC sastavlja objekte i instancira zavisi od metadata u java code tj kako smo mi konfigurisali objekat.

