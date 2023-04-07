# Mongo

Mongo koristi poseban tip koji se zove BSON ,njegov slican tip je JSON.

Bson je jednostavno Binary JSON koji ima ekstenzije za nove podatke,npr ObjectId,TimeStamp,Date

## Komande

<span style="color:red">db</span> -> show current DB

<span style="color:red">show dbs</span>  -> prikazuje sve baze

<span style="color:red">use imeDB</span>  ->pravi novu bazu pod tim imenom

<span style="color:red">db.createCollection("collectionName")</span> -> kreira novu kolekciju

<span style="color:red">db.collection.insertOne({...})</span>  -> u bazi DB u kolekciji collection ubacuje podatak,ako kolekcija ne postoji napravice je

mi kada pisemo ovo db i collection to su imena postojecih baza i kolekcija 

npr:

```
testdb.users.insertOne({...}) 
```



<span style="color:red">db.collection.insertMany({...},{...})</span>

<span style="color:red">db.collection.find()</span> -> ovo je kao getall

<span style="color:red">db.collection.find({...}) </span>-> prikazuje sve koje imaju ovaj objekat kao,nalik njemu

npr:

```
db.collection.find({category:"new"})
```



<span style="color:red">db.collection.find().sort({by})</span> -> sortira po necemo

npr:

```
db.collection.find().sort({title:1}) ->  1 je za ascending a -1 za descending
```



pored sorta postoje i funkcije:

<span style="color:red">count()</span>

<span style="color:red">skip(broj)</span>

<span style="color:red">limit(broj)</span>

<span style="color:red">findOne(obj)</span>



<span style="color:red">$eq</span> -> equality

<span style="color:red">$gt</span> -> greather then

<span style="color:red">$lt</span> -> less then

<span style="color:red">$gte</span> -> greather then equals

<span style="color:red">$lte</span> ->less then equals

<span style="color:red">$in </span>-> in

<span style="color:red">$ne</span> -> not equal

<span style="color:red">$nin</span> -> not in

npr:

```
db.collection.find({likes:{$gt:3}})
```



kada radimo ove operazije moramo da ih grupisemo u {...} 

<span style="color:red">db.collection.updateOne(SearchCriteria,WhatToUpdate)</span> -> update ceo objekat

db.collection.updateOne({title:"T1"},<span style="color:red">$set</span>:{title:"new"}) -> update samo ovo

db.collection.updateOne({title:"T1"},$set:{title:"new"},<span style="color:red">{upsert:true}</span>) -> zadnji parametar su opcije,ovo znaci da ce da insert ako ne postoji

<span style="color:red">db.collection.deleteOne({title:"T1"})</span> ->brise jedan ovo je uslov

<span style="color:red">db.collection.deleteMany(..)</span>





<span style="color:red">$and</span> 

<span style="color:red">$or</span> 

<span style="color:red">$nor</span> 

<span style="color:red">$not</span> 

njih pisemo u [..]

npr:

b.collection.findOne({$and:<span style="color:red">[</span>{query},{query}<span style="color:red">]</span>})

<span style="color:red">$exists</span> -> da li polje postoji

<span style="color:red">$type</span> -> da li je taj type

 npr:

```
db.collection.findOne({address:{$exists:true}})
db.collection.findOne({address:{$type:"double"}})
```



<span style="color:red">$mod:[brojkojimdelim,ostatak]</span>

```
db.collection.find({"quantity":{$mode:[3000,10]}})
```

<span style="color:red">$regex:"..."</span>

```
db.collection.find({name:{$regex:"pattern"}})
```

<span style="color:red">$all:[...]</span>

```
db.collection.find({category:{$all:["healthy","oragnic"]}})
```

<span style="color:red">$size:broj</span>

```
db.collection.find({category:{$size:2}})
```

<span style="color:red">$elemMatch</span> -> makar 1 element u nizu da ispuni

```
db.collection.find({ctegory:{$elemMatch:{$gt:100,$lt:200}}})
```

<span style="color:red">db.dropDatabase()</span>

<span style="color:red">db.collection.drop()</span>

<span style="color:red">db.collection.find({uslov},{projections}) </span>-> ako je taj field 1 on ce se prikazati,a ako je 0 nece

npr:

db.collection.find({uslov},{field:1})



## Index

<span style="color:red">db.column.createIndex({field:type}) </span>

ali  je lakse da napravimo preko MongoCompass ui aplikacije

Tipovi indexa:

- 2D -> koristi Plane
- 2DSphere -> koristi Sphere
- 1/ASC -> rastucie
- -1/DESC -> opadajuce
- Text 



## GeoJson

Posebna vrsta strukture koja radi sa geo lokacijama

<span style="color:red">$geaometry</span>

<span style="color:red">type:</span> -> "Point" ,"LineString","Polygon"

<span style="color:red">coordinates:[...]</span>

<span style="color:red">$maxDistance</span>

<span style="color:red">$minDistance</span>

ovo mora da sadrzi

Da bi radili moramo da imamo index **2DSphere**

<span style="color:red">$near</span>

```
location:{$near:{$geometry:{type:"Point",coordinates:[73,24]},$masDistance:1000,$minDistance:10}}
```

<span style="color:red">$nearsphere</span>

isto postvaljamo kao i $near samo sto ovo kopristi spehere

**geogson.io** sajt

<span style="color:red">$geowithin</span>

```
location:{$geowithin:{$centerSphere:[[coordinates],10]}}
```



## Embeded Query

{	item:"journal".

gty:25,

size:{h:14,w:21,vom:"cm"},

status:"A"

}

```
db.collection.find({size:{h:14,w:21,vom:"cm"}}) -> trazice gde je size obj jedan ovom
```

```
db.collection.find({"size.vom":"cm}) -> ulazi u size i gleda samo vom
```

```
db.collection.find({"size.h":{$et:15},"size.vom":"cm"})
```

## Array Query

{	item:"journal".

gty:25,

tags:["blank","red"],

dim_cm:[14,21]

}

```
db.collection.find({tags:["blank","red"]}) ->mora da ima tacno ova dva elementa i isti redosled
```

```
db.collection.fin({tags:{$all:["blank","red]}}) -> mora da ima sve ove elemente ali redosled nije bitan
```

```
db.collection.find({dm_cm:{$gt:24}}) -> mora da ima bar 1 ele koji je veci od 24
```

```
db.collection.find({"dm_cm.1":{$gt:20}}) -> prvi element niza mora da bude veci od 20 
```

​	$isumber

<span style="color:red">$isNumber:"name"</span>

mora :"Ime var"

```
$type:"number" isto radi
```

## Text Query

Mora da ima text filed i da postoji text index ,koristi se interno **B-Tree**

ne radi ako pisemo samo 

description:"textneki" vec mora drugacije da se kuca

<span style="color:red">$text:</span>

<span style="color:red">{</span>

<span style="color:red">$search:"textneki",</span>

<span style="color:red">$caseSensitive:bool,</span>

<span style="color:red">$language:"String",</span>

<span style="color:red">$diacriticSensitive:bool </span>-> ovo je za apostrof na recima kao u francuskom imamo npr neka lsova sa znacima gore 

<span style="color:red">}</span>

nisu sva ova polja obavezna.

npr:

```
{$test:{$search:"miss"}}
```



## Projections

ovo je da menjamo output query-ja.Imamo dobar ui alat u MongoCompass za rad

<span style="color:red">$project</span> -> da li pokazujemo to na ekranu

**varijabla:broj -**> broj ide ili 1 ili 0 ,tj da li ce se prikazati ili ne

<span style="color:red">$match</span> -> ovo je match za query





$group -> deli kolekciju u grupe na osnovu group key

<span style="color:red">$group:{</span>

<span style="color:red">_id:expression</span> -> po cemu se grupise,group key,kategoricka varijabla

<span style="color:red">field:{accumulator:expression}</span> -> primenu funkcije na grupi

<span style="color:red">}</span>

npr:

**_id:"$field"** vratice samo distinct values,mora sa ovim $ ime







<span style="color:red">$avg</span>

funkcija koju ubacuje u accumnulator delu $group

_id:"title",

average:$avg:"$runtime" 

grupise polja po title a onda radi avg za svaku grupu 

<span style="color:red">$bottom</span>

vraca bottom element u grupi

$bottom:{

output:["field..."],

sortBy:["field:1"]

}

<span style="color:red">$count</span>

$count:{}

ovo je isto agregatna funkcija

_id:"$title",

broj:{

$count:{}

}

<span style="color:red">$first</span>

$first:expression

_id:"$title",

prvi_release:{

**$first**":$release"

}



prvi release za title koji smo grupisali



<span style="color:red">$firstN</span>

$firstN:{

input:expression,

n:expression

}



<span style="color:red">$last</span>

$last:expression

_id:"$title",

last_release:{

**$last**:"$release"

}



<span style="color:red">$lastN</span>

$lastN:{

input:expression,

n:expression

}



<span style="color:red">$max</span>

$max:expression

_id:"$title",

max_m:{

**$max**:"$runtime"

}



<span style="color:red">$min</span>

$min:expression



<span style="color:red">$maxN</span>

$maxN:{

input:expression,

n:expression

}



<span style="color:red">$push</span>



<span style="color:red">$sum</span>

$sum:expression

_id:"$title",

zbir:{

$sum:"$runtime"

}



<span style="color:red">$top</span>

<span style="color:red">$sort</span>

field:1



<span style="color:red">$count</span>













Mi mongo registrujemo u Spring aplikaciji tako sto:

1) Uvezemo biblioteku za mongo

2) Podesimo konfiguraciju

   

```yaml
data:
  mongodb:
    uri:  niki
    database: neka
```

Mi Mongo mozemo da koristimo na 2 nacina:

### Preko Template-a

Osnovni pojmovi:

- <span style="color:red">MongoTemplate</span>
- <span style="color:red">MongoClient</span>

Prvo mormao da napravimo MongoTemplate klasu 

@Bean

```java
@Bean
public MongoTemplate template(){
	return new MongoTemplate(MongoClient,dbName);
}

@Bean
public MongoClient mongoClient(){
    ConnectionString connectionString=new ConnectionString(URL);
    MongoClientSettings settings=MongoClientSettings.builder()
                                                    .applyConnectionString(connectionString)
                                                    .build();
    return MongoClients.create(settings);
}
```

i sada mozemo da autoview ovo.

### Query

Ovo je bitan objekat za mongo kada radimo preko mongo template queries.

```java
Query q=new Query();
q.addCriteria(Critetia)
 .limit(int)
 .maxTime(Duration)
 .skip(int)
 .with(Sort)
```



### Criteria

Vidimo da kada radimo query ,radimo po nekoj kriteriji.

```java
Criteria criteria=Criteria.where(StringKey)
						.is(obj)
						.and(key)
						.all(objects)
						.exists(boole)
						.gt
						.lt
						.gte
						.lte
						.mod
						.near
						.ne
						.not
						regex
```



### Update

```java
Update update=new Update()
	update.set(key,value)
```

 

Mongo Repositories





## Mongo Entity

Ovde nemamo @Entity kao u JPA za relacione baze,vec imamo:

<span style="color:red">@Document(collection)</span>

npr

```java
@Document(collection="products")
public class Product(){

}
```



PK

<span style="color:red">@id</span>

Po defaultu je u mongu primary key ObjectId klasa,ako mi ne kazemo sta je id on ce sam da generise.Tako na primer ako imamo ovu klasu on ce automatski da generise ObjectId

```java
public class Product {
    private String name;
}
```

mi mozemo da stavimo

```java
public class Product {

    private ObjectId id;
    private String name;

}
```

Ovde ce da prepozna sta je kljuc automatski,razlika je u tome sto cemo imati pristup ovom id polju sto je jako bitno,svakako ce ubaciti ObjetId definisali mi to ili ne.

```java
public class Product {

    private String id;
    private String name;

}
```

Ako definisemo ovako,on ce sam da prepozna po imenu da je id primary,tako da ce to biti PK i nece generisati Objectid,ali ako se ne zove id nece ga prepozati,sa strane koda imamo da vidimo kao da li je prepoznalo.Cim se ne zove id nece biti id,i onda ce sam da generise ObjectId

```java
public class Product {

    @Id
    private String test;
    
    private String name;

}
```

Ovako bi forsirali da je to id,kao i u JPA,ovi slucaji su bili da je automatski sve.

<span style="color:red">@Field(name,type)</span>

Mi mozemo da manipulisemo poljima kao u JPA preko @Column ovde je to Field

```java
   @Field(value = "price",targetType = FieldType.DOUBLE)
pricate Double price;
```



<span style="color:red">@Index(unique,IndexDirection,name,expireAfterSeconds)</span>

Index oznacimo na polje,ovo se nece automatski prepoznati vec mi moramo sami da ukljucimo jer je po defaultu false



```properties
spring.data.mongodb.auto-index-creation=true
```

```java
@Indexed(unique = true,direction = IndexDirection.ASCENDING,name = "test",expireAfterSeconds = 2)
```



```java
@CompoundIndexes({
    @CompoundIndex(name = "email_age", def = "{'email.id' : 1, 'age': 1}")
})
```

Ovo je index koji se pruza na vise kolona

<span style="color:red">@TextIndexed</span>

Ovo stavimo na text polje u kodu,i mozemo da raidmo text serch



<span style="color:red">@GeoSpatialIndexed</span>

Ovo je indeks koji stavljamo na polja koja bi trebalo da budu geo lokacije

```java
@Document(collection = "products")
public class Product {

    @Id
    private String test;

    @GeoSpatialIndexed
    double[] rectangle;
}
```



Stavicemo da bude double[] tako mora.

```java
List<Product> findByRectangleWithin(Circle circle);
List<Product> findByRectangleWithin(Box box);
```

Samo imena metoda moraju biti ovakva,i da traze shape objekte



<span style="color:red"> @Transient </span>   

Polje se nece persistovati



<span style="color:red"> @PersistenceConstructor </span>   

Oznacimo konstruktor i to ce mongoTemplate da koristi kada persist objekat

### 
