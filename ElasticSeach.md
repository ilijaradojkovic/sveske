# ElasticSeach



```yml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
    ports:
      - '9200:9200'
    volumes:
      - esdata:/var/lib/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    ports:
      - '5601:5601'
    depends_on:
      - elasticsearch

volumes:
  esdata:
    driver: local


```



- Database
- Datastore

Osobine:

- NoSql
- Distributed
- JSON
- REST interakcija(Znaci mozemo request-response,kao i stream neki)

Ova je `NoSql` baza,i svaka interakcija sa ovom bazom je preko `REST-a`,ona je `distribuirana JSON` baza

Uporedjivanja sa RDBMS:

|  RDBMS   |     ElasticSeach     |
| :------: | :------------------: |
| Database |        Index         |
|  Table   | Index Patterns/Types |
|   Row    |       Document       |
|  Column  |        Fields        |
|          |                      |



![image-20230613155528909](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230613155528909.png)



Kibana -> Web based UI pomocu koga imamo interakciju sa bazom.Mi mozemo da pravimo sami komponente za to sve,widgets,vizualizacije...

LogStash -> Open source server side processing pipeline.Ima dva posla da uzme podatke(input) da ih transformise i da ih stash-uje negde (ovo je kao outpu)



Index je podeljen u `Shards`,

Shard u index-u je podeljen na `Primary` i na `Replica`

![image-20230614094327239](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230614094327239.png)

Node 1 znaci da pokrecemo samo 1 instancu.Mi mozemo da imamo i vise instanci

![image-20230614094635586](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230614094635586.png)

Kada dodje `write request` on se prvo usmerava na primary pa tek onda da replica.`Read request` se usmerava random ili na primary ili na replica

Replica moze postati primary ako primary padne.



Seach Index -> Trazi

Inverted Index -> koliko puta se koja rec ponovila u kojem dokumentu

Term Frequency -> Koliko se cesto rec pojavljuje u datom dokumentu

Document Frequency -> Koliko se cesto rec pojavljuje u svim dokumentima

Term Frequency / Document Frequency -> Dobijemo koliko je rec posebna ,cesta

## Mapping

Ovo je schema definition.

#### Field types

```
"properties":{
	"user_id":{
		"type":"long"
	}
}
```

#### Field Index

for full-text seach (analyzed/not_analyzed/no)

```
"properties":{
	"genre":{
		"index":"not_analyzed"
	}
}
```

#### Field Analyzer

Character Filters -> Remove HTML encoding,cnvert & to and

Tokenized -> Split Strings on whitespace/punction/non-letters

Token Filter -> Lowercasting,stemming,synonyms,stopwords

za analyzer-e imamo izbora na :

- Standard -> splits on word boundaries...
- Simple -> splits anyting that isnt a letter,and lowercase
- Whitespace -> splits any whitespace
- Language ->language-spesific 

```
"properties":{
	"description":{
		"analyzed":"english"
	}
}
```

## API

Jer sam instalirao container u docker-u moramm da pristupim docker comtaineru kako bih izvrsavao CURL komande

```
docker ps -> da viidmo sve container-e
docker -exec -it idContainer bash
```

### Index

Kreiraj index

```curl
curl -XPOST http://localhost:9200/IMEINDEXA

POST /ImeIndeksa
```

with custom shard and replicas 

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/userindex2?pretty -d '{
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas": 1
    }
  }
}'


POST /ImeIndeksa
    "settings":{
        ....
    }
```

Ovo pravi prazan index,on nema strukturu uopste,nema schemu

```
curl -XPUT -H "Content-Type: application/json"  http://localhost:9200/userdetails3?pretty -d '{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
      "properties": {
        "user_id": {
          "type": "keyword"
        },
        "last_login_date": {
          "type": "date"
        },
        "user_name": {
          "type": "keyword"
        }
      }
    }
}'
```

Da bi imao shcemu moramo da dodamo mappings

### Mappings

```
"mappings":{
	"properties":{
		"FiledName":{
		 	"type":"FieldType"
		}
	}
}
```



#### FieldTypes

- binary -> prima Base64 encoded vrednost kao string (U29tZSBiaW5hcnkgYmxvYg==)

- boolean

- completion -> sugestije kao

- date (kako json nema Date kao datatype nema ni elasticseach,ovo je string,internally dates se pretvaraju u UTC),kada sam dodao date onda sam morao lepo da filtriram index u kibni jer nije hteo lepo da radi jer nije u date range tog datuma

  ```
  "mappings": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
  ```

  Ovde mozemo da dodamo i format,on cuva podatke kada konvertuje u `milisekunde`

- date_nanos (On cuva date kao `nanosekunde`)

- flattened (ovo znaci da ne znamo koja je struktura i da ce se ona sama odrediti dinamicki)

  ```
  PUT bug_reports
  {
    "mappings": {
      "properties": {
        "title": {
          "type": "text"
        },
        "labels": {
          "type": "flattened"
        }
      }
    }
  }
  //dodavanje 
  POST bug_reports/_doc/1
  {
    "title": "Results are not sorted correctly.",
    "labels": {
      "priority": "urgent",
      "release": ["v1.2.5", "v1.3.0"],
      "timestamp": {
        "created": 1541458026,
        "closed": 1541457010
      }
    }
  }
  
  ```

- geo_point (latitunde-longitude pair)

  Ovde imamo vise nacina pisanja ,napravimo index

  ```
  PUT my-index-000001
  {
    "mappings": {
      "properties": {
        "location": {
          "type": "geo_point"
        }
      }
    }
  }
  ```

  ```
  PUT my-index-000001/_doc/1
  {
    "text": "Geopoint as an object using GeoJSON format",
    "location": { 
      "type": "Point",
      "coordinates": [-71.34, 41.12]
    }
  }
  ```

  ```
  PUT my-index-000001/_doc/3
  {
    "text": "Geopoint as an object with 'lat' and 'lon' keys",
    "location": { 
      "lat": 41.12,
      "lon": -71.34
    }
  }
  ```

  ```
  PUT my-index-000001/_doc/4
  {
    "text": "Geopoint as an array",
    "location": [ -71.34, 41.12 ] 
  }
  ```

  ```
  PUT my-index-000001/_doc/5
  {
    "text": "Geopoint as a string",
    "location": "41.12,-71.34" 
  }
  ```

- geo_shape(ovo je oblik samo,kao pravougaonik,poligon,linije)

  ```
  POST /example/_doc
  {
    "location" : {
      "type" : "LineString",
      "coordinates" : [[-77.03653, 38.897676], [-77.009051, 38.889939]]
    }
  }
  ```

  ```
  POST /example/_doc
  {
    "location" : {
      "type" : "Polygon",
      "coordinates" : [
        [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0] ]
      ]
    }
  }
  ```

  ```console
  POST /example/_doc
  {
    "location" : {
      "type" : "Polygon",
      "orientation" : "LEFT",
      "coordinates" : [
        [ [-177.0, 10.0], [176.0, 15.0], [172.0, 0.0], [176.0, -15.0], [-177.0, -10.0], [-177.0, 10.0] ]
      ]
    }
  }
  ```

- ip(cuva ip adresu)

- join(kreira parent - child vezu)

- keyword (uzima samo unique vrednosti,obicno koristimo za id,email,status code...)

  ```
  PUT idx
  {
    "mappings": {
      "_source": { "mode": "synthetic" },
      "properties": {
        "kwd": { "type": "keyword" }
      }
    }
  }
  PUT idx/_doc/1
  {
    "kwd": ["foo", "foo", "bar", "baz"]
  }
  
  sacuvace:
  {
    "kwd": ["bar", "baz", "foo"]
  }
  
  ```

  ```
  PUT idx
  {
    "mappings": {
      "_source": { "mode": "synthetic" },
      "properties": {
        "kwd": { "type": "keyword", "store": true }
      }
    }
  }
  PUT idx/_doc/1
  {
    "kwd": ["foo", "foo", "bar", "baz"]
  }
  sacuvace: -> stavili smo store na true
  {
    "kwd": ["foo", "foo", "bar", "baz"]
  }
  ```

  ```
  PUT idx
  {
    "mappings": {
      "_source": { "mode": "synthetic" },
      "properties": {
        "kwd": { "type": "keyword", "ignore_above": 3 }
      }
    }
  }
  PUT idx/_doc/1
  {
    "kwd": ["foo", "foo", "bang", "bar", "baz"]
  }
  sacuvace: -> Ovde je bitan redosled jer ovo ignore_above ce prve 3 stvari stavii na kraju
  {
    "kwd": ["bar", "baz", "foo", "bang"]
  }
  
  ```

  - constant_keyword(konstanta,prati je polje value )

    ```
    PUT logs-debug
    {
      "mappings": {
        "properties": {
          "level": {
            "type": "constant_keyword",
            "value": "debug"
          }
        }
      }
    }
    ```

  - wildcard(dugacki stringovi nad kojima radimo regex)

  - nested(ovo ubacujemo niz drugih objekata)

    ```
    PUT my-index-000001/_doc/1
    {
      "group" : "fans",
      "user" : [ 
        {
          "first" : "John",
          "last" :  "Smith"
        },
        {
          "first" : "Alice",
          "last" :  "White"
        }
      ]
    }
    ```

  - long

  - integer

  - short

  - byte

  - double

  - float

  - object(ovo je inner object)

  - point(prima niz x,y koordinata)

  - integer_range

  - float_range

  - long_range

  - double_range

  - date_range

  - ip_range

  - text
  
  - arrays -> U es nema kao konkretan tip podataka da je array,vec svako polje moze da ima vise vrednosti,ali one moraju biti istog tipa















Vidi index

```
curl -XGET http://localhost:9200/IMEINDEXA

curl -XGET http://localhost:9200/IMEINDEXA?pretty -> ovo ce da formatira JSON

GET /INDEXNAME
```

List all indexes

```
curl -XGET http://localhost:9200/_cat/indices?v

GET /_cat/indices
```

List index mappings (koja polja ima index)

```
curl -X GET http://localhost:9200/INDEXNAME

GET /INDEXNAME
```

Delete Index

```
curl -X DELETE http://localhost:9200/INDEXNAME

DELETE /INDEXNAME
```

Add data

Index mora da ima strukturu da bi dodali nesto

ili cemo jednostavno da dodamo nesto i on ce sam da ga update i da kreira document 

```
curl -XPUT http://localhost:9200/movies/_doc/109487 -H "Content-Type:application/json" -d '{"genre":["IMAX","Sci-Fi"],"title":"Interstellar","year":2014}'


PUT /movies/_doc/123
{
  "id":"123",
  "title":"test1",
  "year":2020,
  "genre":["Action"]
}
```

bitan je url

znaci mi u index movies(ako ne postoji napravice ga sa semom u body sto mu damo),u dokumentu sa tim id-jem(ako ne postoji napravice ga)

### Document 

Kreiraj document

```
curl -XPOST http://localhost:9200/movies/_create/109487 -H "Content-Type:application/json" -d '{"genre":["IMAX","Sci-Fi"],"title":"Interstellar","year":2014}'

PUT /movies/_create/1234
{
"genre":["IMAX","Sci-Fi"],"title":"Interstellar","year":2014
}

```

```
curl -XPUT http://localhost:9200/movies/_doc/109487 -H "Content-Type:application/json" -d '{"genre":["IMAX","Sci-Fi"],"title":"Interstellar","year":2014}'

PUT /movies/_doc/1234
{
"genre":["IMAX","Sci-Fi"],"title":"Interstellar","year":2014
}
```

For je da u prvoj imamo `_create` a u drugoj imamo `_doc` i to nije isto jer je _doc update i create a _create ce da fail ako vec postoji dokument sa tim id-jem

Mi ovde ubacujemo jedan po jedan dokument,ali moguce je i vise njih preko `_bulk` operatora

```
curl -H "Content-Type:application/json"-XPUT http://localhost:9200/_bulk -d '
{
	{"create":{"_index":"movies","_id":"123112"}}
	{"id":123112,"title":"test","year":2000},"genre":["Action"]}

	{"create":{"_index":"movies","_id":"123112"}}
	{"id":123112,"title":"test","year":2000},"genre":["Action"]}
	
		{"create":{"_index":"movies","_id":"123112"}}
	{"id":123112,"title":"test","year":2000},"genre":["Action"]}

}

'
```

fora je sto imamo liniju json objekta za create koji mu kaze u kom indeksu i koj id dokumenta

Document je immutable,on ima _version polje pa mozemo imati vise verzija istog dokumenta pa ce es koristiti najnoviji,tako da kada update on ce napraviti potpuno novi dokument sa novom _version a stari ce biti mark za delete

Update document

koristimo _update u putanji

```
curl -H "Content-Type:application/json"-XPOST http://localhost:9200/movies/_update?109487 -d '{
"doc":{ -> mora doc ,bukv taj naziv pa u njega idu variables
	"title":"Interstellar"
}
}'

POST /movies/_update/109487
{
"doc":{ 
	"title":"Interstellar"
}
}
```

fora je sto mi mozemo da update i sa ovim  _doc ,samo sto bi morali da ubacimo ceo body,ako nesto fali on ce ga staviti na null,dok sa _update mi samo stavimo ono sto nama treba za update

```
 curl -H "Content-Type:application/json" -XPUT http://localhost:9200/movies/_doc/109482?pretty -d '{"genre":["IMAX"],"title":"test","year":2021}'
 
 
 
```

Delete document

```
curl  -XDELETE http://localhost:9200/movies/_doc/ID

DELETE /INDEXNAME/_doc/ID
```

Search document

```
curl -XGET http://localhost:9200/movies/_search?q=Dark
```

## Concurency Problem

![image-20230614124352121](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230614124352121.png)

vidimo da 2 klijenta dobijaju isti broj 10,ali kada ga upadate svaki pojedinanco to radi,treba da bude 12 jer su 2 klijenta update,ali zbog tog citanja i nesinhronizacije dobija se pogresno

Resenje za ovo je `Optimistic Concurency Control`

Ovo smo nesto slicno imali kod update, imali smo _version polje.Ovde imamo `_sequance number` i



![image-20230614125121002](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230614125121002.png)



VIdimo da kada zahtevamo broj on vraca `_seq_no` i `_primarty_term` zajedno sa podacima,ako vise njih okusava da update sa istim tim podacima onda se desava greska ,samo 1 moze to da uradi i time imamo sinhronizaciju

mi ako get 

```
 curl -XGET http://localhost:9200/movies/_doc/12412?pretty
```



vidimo da imamo _seq_no i _primary_term ,i to imamo za svaki document

![image-20230614125201685](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230614125201685.png)

mi kada update _seq_num ce se promeniti pa onaj ako update u isto vreme nece imati isti _seq_num,zato pri pisanju requesta treba ovo da uzmemo u obzir

klasican update:

```
curl -H "Content-Type:application/json"-XPOST http://localhost:9200/movies/_update/109487 -d '{
"doc":{ -> mora doc ,bukv taj naziv pa u njega idu variables
	"title":"Interstellar"
}
}'
```

Ovde ne specificiramo seq num pa nece ni da gleda,samo ce da update nezavisno od toga,i tu se stvara problem 

sa _seq_num

```
curl -H "Content-Type:application/json"-XPOST http://localhost:9200/movies/_update/109487?pretty=true&if_seq_no=10&if_primary_term=1 -d '{
"doc":{ -> mora doc ,bukv taj naziv pa u njega idu variables
	"title":"Interstellar"
}
}'
```

ovo je bitno `if_seq_no=10&if_primary_term=1` mi kazemo da hocemo da se updae za te kljucne info,ali ako je update neko vec onda nece biti ovi brojevi pa ce da baci gresku,ovo je vise safe

## Analyzers

oni analiziraju text kada se radi seach

Imamo 2 opcije kod seach-a

1. Exact-Match

   Kod exact-match moramo da koristimo keyword mapping instead of text ?

2. Partial

   koristimo analyzere,i koji analyzeri ce se izvrsiti na kojim poljima

   `match`

```
curl -H "Content-Type:application/json" -XGET http://localhost:9200/movies/_search?pretty -d '{ "query":{"match":{"title":"Star Trek"}}}'
```

 vratice nam Star Wars i Star Trek filmove,ovo je partial deo je match jer se star match,ali je score za starwars manji,svejedno ga je nasao.

`match_phrase`

```
curl -H "Content-Type:application/json" -XGET http://localhost:9200/movies/_search?pretty -d '{ "query":{"match_phrase":{"genre":"sci"}}}'
```

Sada cemo da dodamo analyzer,on se dodaje u mappings kada pravimo index

```
curl -XPUT -H "Content-Type: application/json"  http://localhost:9200/userdetails3?pretty -d '{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "sampletype": {
      "properties": {
        "user_id": {
          "type": "keyword"
        },
        "last_login_date": {
          "type": "keyword"
        },
        "user_name": {
          "type": "keyword",
          "analyzer":"neki"
        }
      }
    }
  }
}'
```

Vidimo polje user_id koji je tipa `keyword` i onda se ono ne analizira tj ne primejuje se default analyzer vec moramo exact da ga pogodimo (case sensitive) 

Postoje vec definisani analyzeri npr

1. english

## Data Modeling

### Inned Objects

Ovo je dobro kada imamo one-to-one mapping.Oni se ne indeksiraju posebno vec kao deo parent objekta.Tretiraju se kao bilo koji field pa je query kao i za svaki drugi field isti

```json
{
  "name":"Zach",
  "car":{
    "make":"Saturn",
    "model":"SL"
  }
}
```



### Nested Objects

Ako unesemo jos jedan objekat onda ovo inner objects nece imati smisla 

```
{
  "name" : "Zach",
  "car" : [
    {
      "make" : "Saturn",
      "model" : "SL"
    },
    {
      "make" : "Subaru",
      "model" : "Imprezza"
    }
  ]
}
{
  "name" : "Bob",
  "car" : [
    {
      "make" : "Saturn",
      "model" : "Imprezza"
    }
  ]
}
```

ovako (inner) nece da radi dobro pa se moramo prebaciti na nested

Nested objekti sadrze listu objekata,svaki element niza se tretira kao poseban dokument,pa se query radi na specijalan nacin

```
{
  "name" : "Zach",
  "car" : [
    {
      "make" : "Saturn",
      "model" : "SL"
    },
    {
      "make" : "Subaru",
      "model" : "Imprezza"
    }
  ]
}
```

Takodje pri pravljanju indeksa moramo posebno da ih definisemo sa `nested` tipom

```
{
  "person":{
    "properties":{
      "name" : {
        "type" : "text"
      },
      "car":{
        "type" : "nested"
      }
    }
  }
}
```

Glavna razlika sa inner jer sto se ne tretiraju kao pojedinacni dokumenti



### Parent/Child

ovo je vise loose coupling od nested 

Ovako ce nam izgledati parent



```

Person
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

Napravicemo jedan dokument za person da bi imali sa cim da povezemo 

```
PUT /my_index/_doc/1
{
  "title": "Parent Document"
}
```

dok ce child 

Kreirace odmah i index 

```
PUT /my_index/_doc/2?routing=1&refresh=wait_for
{
  "title": "Child Document",
  "parent": "1",
  "salary": 50000
}
```



glavna fora je sto kada budemo radili ovde sad sa curl-om moramo da kazemo ?parent=idtogparenta

Zelimo parent-child relaciju

Franchine(Parent) i Film(Child) ce biti u toj vezi.

```
curl  -H "Content-Type: application/json" -XPUT http://localhost:9200/series -d '
{
	"mappings":{
		"properties":{
			"film_to_franchise":{
				"type":"join",
				"relations":{"franchise":"film"}
			}
		}
	}
}'
```

 

## Search

#### Filters

```
GET /my_index/_search
{
  "query": {
  		"bool":{
  			"must":[{}],
  			"filter":[{}],
  			"must_not":[].
  			"should":[],
  			
  		}
  	}
 }

```

**term** -> mora da match ovu vrednost

```
{"term":{"year":2014}}
```

**range** -> da se to polje nalazi u range

```
{"range":{"year":{"gte":2010}}}
```

**exist** -> da postoji

```
{"exists":{"field":"tags"}}
```

**terms** ->ako se bilo sta poklopi u listi vrednosti 

```
{"terms":{"genre":["Sci-Fi","Adventure"]}}
```

**missing** -> pronadji dokument gde filed ne postoji

```
{"missing":{"field":"tags"}}
```

**bool**

**mush** -> znaci da ce morati da ispuni ovaj uslov(AND)

**must_not** -> da ne ispuni ovo(NOT)

**should** -> makar 1 od uslova mora biti ispunjen(OR)

```
GET /my_index/_search
{
  "query": {
  		"bool":{
  			"must":[
  			{
                      "term": {
                        "field1": "value1"
                      }
                    },
                    {
                      "range": {
                        "field2": {
                          "gte": 100,
                          "lte": 200
                        }
                      }
        }],
  			"filter":[{}],
  			"must_not":[  {
                  "exists": {
                    "field": "field4"
                  }
        }].
  			"should":[],
  			
  		}
  	}
 }
```

### Queries

**match**

```
GET /my_index/_search
{
  "query": {
  		"match":{
  			"title":"star"
  		}
  	}
 }
```

**match_all**

```
{"match_all":{}}
```

**match_phrase**

```
{"match_phrase":{"title":"star wars"}}

{"match_phrase":{"title":"star beyond","slop":1}} -> ovo slop znaci da moze neka rec samo jedna jer smo stavili 1 da se nalazi izmedju njih ili oko njih,pronaci ce star beyond,start wars beyond,star treck beyon kao da dozovoljava da neka rec upadne i da ovo start beyond mora da se nalazi ali redosled je nebitan
```



## Pagination

koristimo parametre 

**from**(page) i **size**

```
GET /movies/_search
{
  "from": 0,
  "size": 1, 
  "query": {
  		"match":{
  			"title":"star"
  		}
  	}
 }
```

## Sorting

ubacimo u query param sort=ime

```
GET /movies/_search?sort=year
```

imamo problem ovde,jer ne mozemo da search na analyzed filed.Da bi ovo resili moramo da napravimo raw verziju tog fielda

```
{
"mappings":{
	"properties":{
		"title":{
			"type":"text"
		}
	}
}
}
ovako smo normalno pravili i on je tipa text sto ce se analizirati i time mi ne mozemo da sort
--------------------------------------------------------
{
"mappings":{
	"properties":{
		"title":{
			"type":"text",
			"fields":{
					"raw":{
						"type":"keyword"
					}
			}
		}
	}
}
}
ovime smo napravili inner field u polju title.Ovime pristupamo preko title.raw
```

## Fuzzy search(Teorija)

Search odprili tu rec,ta rec mozda sadrzi greske,nedostaje slovo,promasili smo slovo itd

Instaliracemo preko docker-a

ovo se zove i **levenshtein edit distance**

postoje 3 vrste

1. Subsitutions (interstellar -> intersteller) ako se postuje ova distanca od 1 jer je 1 slovo greska
2. Insertions (interstellar->insterstellar)
3. Deletion (intestellar ->interstelar) 

mi specificiramo koliko cemo da tolerisemo

po defaultu je 0.5

```
{
	"query":{
		"fuzzy":{
			"title":{
				"value":"intresteller","fuzziness":2
			}
		}
	}
}
```

## Prefix Query

prefix ce traziti za bilo koju godinu koja pocinje sa 201...

```
{
	"query":{
		"prefix":{
			"year":"201"
		}
	}
}
```

## WildCard Query

koristimo wildcard nacin,ovde kaze da je ovo * bilo sta

znaci 

```
{
	"query":{
		"wildcard":{
			"year":"1*"
		}
	}
}
*1*-> sadrzi 1
```

## Search while you type

Ovo radi samo,samo sto treba da povecamo slop vrednost.Fora je samo da koristimo ovo prefix jer nas zanima uvek pocetak,mada to je sada do asmih specifikacija

```
{
	"query":{
		"match_phrase_prefix":{
			"title":{
				"query":"star trek",
				"slop":10
			}
		}
	}
}
```

Tako nekako funkcionise,ali postoji poseban filed type za ovo

```
PUT /jobs
{
  "mappings": {
    "properties": {
      "title":{
        "type": "search_as_you_type"
      }
    }
  }
}
```

Definisemo polje  **search_as_you_type**

automatski pravi inner filed

```
 _2gram 
_3gram 
_index_prefix
```



### N-Gram

```
"star"

unigram: [s,t,a,r]
bigram: [st,ta,ar]
trigram [sta,tar]
4-gram [star]
```

sto nama ovo treba?

po dafaultu radi ovaj standard analyzer i on

```
GET /movies/_search
{
  "query": {
    "match": {
      "title": {
          "query": "star tr"
          , "analyzer": "standard"
      }
    }
}
}
```

ovde ce da nam vrati 

```
star treck 
star wars 
```

ali star wars uopste nema veze sa ovim,zato nam treba ovo n-gram,jer standard analizira reci pojedinacno i ako ima hit na jednu makar onda stavlja u rezultat

moramo da napravimo taj analyzer i da ga primenimo

```
PUT /movies
{
	"settings":{
		"analysis"{
			"filter":{
				"autocomplete_filter":{
					"type":"edge_ngram",
					"min_gram":1,
					"max_gram":20
				}
			},
			"analyzer":{
				"autocomplete":{
					"type":"custom",
					"tokenizer":"standard",
					"filter":[
						"lowercase",
						"autocomplete_filter"
					]
				}
			}
		}
	}
}
```

## Import Data

## Logstach

On se nalazi izmedju nasih podataka i gde mi zelimo da ih smestimo

![image-20230615153527674](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230615153527674.png)

## Agregations

Ovo je slicno kao u mongu

```
GET /movies/_search
{
  "aggs": {
    "NAME_AGGREGATION": {
      "TYPE": {}
    }
  }
}
```

Ovo je sintaksa

### terms

```
GET /movies/_search
{
  "aggs": {
    "years": {
      "terms": {
        "field": "year"
      }
    }
  }
}
```

grupisace godinu i koliko ih ima po grupi

### histograms

Onaj grafik kao bar diagram,ovo je samo vrsta agregacije

```
{
  "aggs": {
    "year_histogram": {
      "histogram": {
        "field": "year",
        "interval": 700
      }
    }
  }
}
```

### date_histograms

```
{
  "aggs": {
    "year_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": hour
      }
    }
  }
}
```

## Nested Agregations

Ovo su agregacije u agregacijama

npr: average rating for each Star Wars movie

```
GET /movies/_search
{
  "aggs": {
    "title_term": {
      "terms": {
        "field": "title.raw"
      },
      "aggs": {
        "avg_year": {
          "avg": {
            "field": "year"
          }
        }
      }
    }
  }
}
```

unutar jedne agregacije samo napisemo drugu

## Using SQL

Mozemo da koristimo sql 

```
POST /_sql?format=txt
{
"query":"your sql"
}
```

## Transform Data



pivot.group_by -> sta cemo da grupisemo

pivot.aggregate -> funkcija agregacije

## Spring ElasticSearch

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
  </dependency>
```

Ovde imamo izbora da radimo preko:

1. ElasticsearchRestTemplate
2. Repositories
3. ElasticSearchClient(Moramo da dodatno uradimo config)



### Anotacije modela

1. <span style="color:red">@Document(indexName)</span>

2. <span style="color:red">@Field()</span>

   ```==
   type=FieldType.neki -> kog tipa je u elasticsearchu
   name="", -> koje je ime varijable
   index=bool, -> da li se polje indeksuje
   store=bool, -> da li se storuje
   analyzer="koj", -> koji analyzer se koristi
   format=DATEFORMAT,pattern="", -> koji format se koristi za date
   fielddata=FieldData.neki -> ovo je za sortiranje i pretrazivanje
   searchAnalyzer=""(default je "") -> ovo je analyzer za search 
   normalizer=""(default je "") -> 
   coerce=bool (default je true) ->konvertovanje vrednosti u tip elasticseach-a
   ignoreAbove=int(deafult je -1) ->
   ignoreMalformed=bool(default false) -> skip ce invalid vrednosti za ovaj field
   includeInParent=bool(default je false) -> da ukljuci roditelja pri prikazu ovih podataka
   storeNullValues=bool(defaule je false) -> da cuva null vrednosti
   enabled=bool(default je true) 
   storeEmptyValues=bool(default je true) -> da cuva empty vrednosti
   ```

   

3. <span style="color:red">@Setting()</span>

   ```
   settingsPath =string -> namestamo 
   useServerConfiguration=bool
   shards =int-> (default je 1)
   replicas=int ->(default je 1)
   refreshInterval =string(default je 1s)
   indexStoreType =string(default je fs)
   sortFields=String[](default je {})
   sortOrders=SortOrder[] (default je {})
   sortModes=SortMode[] (default je {})
   sortMissingValues =SortMissing[](default je {})
   ```

   

## Imam vec neki sample data,i kako sada to spring da parsuje?

Ako imamo podatke samo napravimo odredjenu klasu i bitno je da je oznacimo sa @Document(imeindeksa u es) i da unesemo koja polja hocemo da ima ,on ce sam da ih mapira.Bitno je da poklopimo ime i tip

## Capacity Managment

- Multiple indices as a scaling strategy b

- policy

  Mozemo da definisemo poliku lifecycle index-a

   HOT->WARM->COLD->FROZEN->DELETE

  ```
  PUT _ilm/policy/datastream_policy
  {
  	"policy":{
  		"phases":{
  			"hot":{
  				"actions":{
  					"rollover":{
  						"max_size":"50GB",
  						"max_age":"30d"
  					}
  				}
  			},
  			"delete":{
  				"min_age":"90d",
  				"actions":{
  					"delete":{}
  				}
  			}
  		}
  	}
  }
  ```

  

sada moramo da povezemo ovaj policy za index template

```
PUT _template/datastream_template
{
	"index_patterns":["datastream-*"],
	"settings":{
		"number_of_shards":1,
		"number_of_replicas":1,
		"index.lifecycle.name":"datastream_policy",
		"index.lifecycle.rollover_alias":"datastream"
	}
}
```

## Arhitektura 

U smislu arhitekture imacemo poseban mikroservis koji ce da cuva duplikate podataka u es,cuvacemo znaci u bazi gde su nam stvarni podaci,i u es za pretragu

Ako imamo update,delete,insert to mormoa da uradimo duplo

Ako imamo vec podatke  pa hocemo da uvedemo es moramo da pokrenemo skriptu koja ce sve da ubaci

## Docs

https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html
