# Resource Server

Kako *Resoruce Server* zna da validira token koji mu stigne od client-a.Client je taj token dobio od Authorization Server-a.

Svaki token sadrzi *Signature*,samo onaj ko ima private key ce moci da dekriptuje podatke,dok public key (koji resource server ima) ce moci samo da validira token,ali ne da ga dekriptuje

![image-20230207154355013](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230207154355013.png)

Resource server ce da ima public key,ali nece ga drzati lokalno niti ce se njemu slati ceo key koa file jer ce to stvarati problem pri rotaciji kljuca,vec ce resource server da gadja rutu authorization server-a da dobije public key



Ovaj endpoint nalazimo tako sto gadjemo rutu gde su svi endpointi nase authorization server-a i trazimo jwks_uri



Authorities koji token sadrzi koji Authorization Server vrati su SCOPE_kojscope ovo je dafault nacin



### Opaque Tokens

Kada radimo sa ovim tokenima mi nemamo informacije u njima pa se ovaj princip radi preko ***introspection*** to je proces da backend tj resoruce server dobije token(opaque) i da njega posalje na auth server koji ga validira 

![image-20230207162527279](C:\Program Files\Typora\resources\Docs\img\image-20230207162527279.png)

MI smo pozivom na introspection endpoint koji moze ovaj token da desifruje i da nam da info

Odemo da vidimo sve rute auth servera i nadjemo introspection(Zove se pomocu post-a)

1)moze da gateway bude resource server a mikrosevici ne (tu da se oni osiguraju koristi se service mesh)

2)moze d  svaki mikroservice ima resource server