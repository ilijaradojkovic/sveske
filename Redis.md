# Redis

Redis je baza koja ima karakteristike:

- no-sql
- key-value
- koristi se za cache dosta

Imamo 2 mogucnosti kako da koristimo redis kao no-sql bazu:

1. Intaliramo lokalno
2. Run preko dockera

Takodje imamo i 2 mogucnosti kako cemo pristupiti toj bazi:

1. CLI
2. GUI (Another redis desktop manager)

Redis je **binary safe** sto znaci da je key ili value koji se cuvaju mogu biti bilo sta

Redis radi sa nekim predefinisam kolekcijama:

- List
- Set
- Sorted set
- Map



###  CLI komande:



<span style=color:#DC4C4C>SET KEY VALUE</span>

O(1) operacija koja za neki key namesta vrednost

npr: SET  1 Iiija



<span style=color:#DC4C4C>GET KEY</span>

O(1) operacija koja na osnovu kljuca vraca vrednost

npr: GET 1



<span style=color:#DC4C4C>DEL KEY</span>

O(1) operacija koja na osnovu kljuca brise vrednost

npr: DEL 1



<span style=color:#DC4C4C>BGSAVE</span>

Ovo znaci da sacuva vrednosti,jer je redis in-memory db moramo joj reci da sacuva vrednosti ili ce se ugasiti ako iskljucimo redis server



<span style=color:#DC4C4C>INFO MEMORY</span>

Daje informacije o memoriji



<span style=color:#DC4C4C>INCR KEY</span>

inkrementuje zadat kljuc ali samo ako je on integer

npr: INCR 1 -->2



<span style=color:#DC4C4C>INCRBY  KEY VALUE</span>

inkrementuje zadat ljuc za value puta

npr: INCRBY 1 2 -->3



<span style=color:#DC4C4C>DECR KEY </span>





<span style=color:#DC4C4C>DECRBY KEY VALUE</span>



<span style=color:#DC4C4C>FLUSH ALL</span>

Brise sve kljuceve



SET KEY VALUE <span style=color:#DC4C4C>EX/PX TIME</span>

EX->Seconds

PX->Miliseconds

Postavlja ovaj key da vazi samo za odredjeno vreme

npr:SET 1 ilija EX 2

ovo ce vaziti samo 2 sekunde

<span style=color:#DC4C4C>TTL KEY</span>

time to live,vidi koliko jos kljuc zivi 



SET KEY VALUE <span style=color:#DC4C4C>XX/HX</span>

XX->ovo je update value ali samo ako key posotji

HX->isto kao XX samo sto ako ne postoji napravice ga



#### Liste

<span style=color:#DC4C4C>LPUSH KEY VALUES</span>

<span style=color:#DC4C4C>RPUSH KEY VALUES</span>

Dodaje vrednosti na levoj ili desnoj strani liste

Ovo mozemo gledati kao da za svaki key imamo listu vrednosti

npr: LPUSH 1 1 2 3 4 5

Dodacemo key 1 sa listom [1,2,3,4,5]



<span style=color:#DC4C4C>LRANGE KEY START_INDEX END_INDEX</span>

Vraca range elemenata,ovo je get

LGET 1 0 2 ->[1,2,3]

LGET 1 0 -1-> ako stavimo da je zadnji element -1 znaci idi do kraja ->[1,2,3,4,5]



<span style=color:#DC4C4C>LPOP KEY</span>

<span style=color:#DC4C4C>RPOP KEY</span>

Izbacuje listu



### Set

<span style=color:#DC4C4C>SADD KEY VALUES</span>

Dodaje vrednosti u set 

npr: SADD k1 1 2 3 4 5 -> napravice se [1,2,3,4,5]



<span style=color:#DC4C4C>SMEMBERS KEY </span>

Vraca set



<span style=color:#DC4C4C>SISMEMEBER KEY VALUE</span>

Da li u setu sa KEY postoji VALUE



<span style=color:#DC4C4C>SMOVE KEY_SET1 KEY_SET2 VALUE</span>

Premesta vrednost iz set 1 u set 2



<span style=color:#DC4C4C>SPOP KEY COUNT</span>

Brise 2 elementa u setu sa KEY

npr: SPOP k1 2 ->brise 2 elementa iz k1



<span style=color:#DC4C4C>SREM KEY VALUE</span>

Brise specificnu vrednost u setu



### Map

![Group 234](C:\Users\radoj\Downloads\Group 234.png)



Map je malo kompleksniji da se razume,pa ova slika pomaze,imamo key koji je vezan za radis i objekat map koji je vezan za taj key,a taj objekat key ima svoje keys i values



<span style=color:#DC4C4C>HSET KEY_REDIS (MAP_KEY,MAP_VALUE)</span>

npr:

HSET k1 age 27 name ilija

napravice map koji sadrzi k1-> 

​													  age -> 27

​										             name -> ilija

<span style=color:#DC4C4C>HGETALL REDIS_KEY</span>



<span style=color:#DC4C4C>HGET KEY FIELD</span>



<span style=color:#DC4C4C>HLEN</span>

Vraca length 



<span style=color:#DC4C4C>HEXISTS KEY MAP_KEY</span>

1->Postoji

0->Ne postoji





Sorted Set

Ovo radi na osnovu SCORE vrednosti,po njoj soritra vrednosti,ne automatski



<span style=color:#DC4C4C>ZADD KEY SCORE VALUE</span>

Dodaje u sorted set

 

<span style=color:#DC4C4C>ZRANGE KEY START_INDEX END_INDEX </span>

Vraca elemente



<span style=color:#DC4C4C>ZPOPMIN KEY NUMBER_OF_ELEMENTS</span>

Izbacuje min elemente n puta



<span style=color:#DC4C4C>ZPOPMAX KEY NUMBER_OF_ELEMENTS</span>

Izbacuje maxelemente n puta





