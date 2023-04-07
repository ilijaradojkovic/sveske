# Core Java

##  java.net.http

Ovo je abstract class i sluzi za slanje requetova preko http protokola

Ova klasa se pravi pomocu builder-a

Kada se napravi pomocu builder-a onda je immutable

Za slanje zahteva koristimo 2 metode:

- send(HttpRequest,BodyHandler)
- sendAsync(HttpReques,BodyHandler) -> vratice CompleatableFuture

Ostale metode builder-a su:

- version(Version.neka)
- fallowRedirects(Redirect.neki)
- connectTimeout(Duration)
- authenticator(Authenticator.neki)



### BodyHandlers

BodyHandlers.nesto

- ofString() ->String
- ofByteArray() ->byte[]
- ofFile(path) -> path
- ofLines() -> stream<String>
- discarding()->void



### HttpRequest

Pravimo ga pomocu builder-a

- .get()
- .delete()
- .post(BodyPublisher)
- .uri(Uri)
- .timeout(Duration)
- .header(name,value)

### BodyPublisher

BodyPublisher.nekia

## Future

Ovo je neka vrednost koja ce se vratiti u buducnosti,obicno je koristimo za async operacije

## CompleatableFuture

Ovo je nadogradnja na Future objekat,mozemo da **nadovezujemo** neke akcije

- .thenApply(...)
- .thenApplyAsync(...)
- .thenRun(...)
- .whenCompleate(...)

## IO

Ovo je input outpu paket u javi koji se koristi za input output operacije kao npr za fajlove

Delimo ih na 2 dela

1. **Byte reading/writing**
2. **Character reading/writing**

### Byte

- <span style="color:#ff6347">InputStream</span>
- <span style="color:#ff6347">OutputStream</span>

Ovo su kljucne apstraktne klase koje su bazicne za ostale 

- <span style="color:#ff6347">FileInputStream</span>

- <span style="color:#ff6347">FileOutputStream</span>

  

  Sluze kao omotaci i pruzaju vise funkcionalnosti

- <span style="color:#ff6347">BufferedInputStream </span>

- <span style="color:#ff6347">BufferedOutputStream</span>

Isto omotaci koji pisu i citaju primitivne tipove podataka direktno preko funkcija writeInt() writeBoolean()...

- <span style="color:#ff6347">DataOutputStream</span>
- <span style="color:#ff6347">DataInputStream</span>

Isto omotaci koji pisu i citaju ali to rade sa objektima sada

- <span style="color:#ff6347">ObjectInputStream</span>
- <span style="color:#ff6347">ObjectOutputStream</span>
- <span style="color:#ff6347">PrintStream</span>

### Character

Aostraktne klase koje su bazicne i koje ostale prosiruju

- <span style="color:#ff6347">Reader</span>
- <span style="color:#ff6347">Writter</span>



Sluze za rad sa fajlovima

- <span style="color:#ff6347">FileReader</span>
- <span style="color:#ff6347">FileWritter</span>
- <span style="color:#ff6347">BufferedReader</span>
- <span style="color:#ff6347">BufferedWritter</span>
- <span style="color:#ff6347">PrintWritter</span>





## NIO

Ovo je new input output java paket koji je dosao u novoj verziji jave.

Ovo su **non-blocking** operacije

Ovaj paket koristi bafere i kanale za rad sa fajlovima

Glavni koncepti:

- <span style="color:#ff6347">Buffer</span>
- <span style="color:#ff6347">Channel</span>
- <span style="color:#ff6347">Path</span>



### Path

Za uzimanje putanja do fajlova koristimo:

- <span style="color:#ff6347">Path</span>

- <span style="color:#ff6347">Paths</span>

  

  ```java
  Path.of(...)
  Paths.get(...)
  ```

### Buffer

Apstraktna klasa za sve bafere

- <span style="color:#ff6347">ByteBuffer</span>
- <span style="color:#ff6347">IntBuffer</span>
- <span style="color:#ff6347">LongBuffer</span>
- <span style="color:#ff6347">DoubleBuffer</span>
- <span style="color:#ff6347">ShortBuffer</span>
- <span style="color:#ff6347">MappedButeBuffer</span> ->Mapira ceo buffer u channel i obrnuto

```java
BufferNeki.alocate(bytes) ->ovo vraca buffer 
```



### Channel

Ovo je apstraktna klasa svih channel klasa

- <span style="color:#ff6347">FileChannel</span>
- <span style="color:#ff6347">SocketChannel</span>

Mozemo dobiti referencu channel klase na vise nacina:

1. <span style="color:#ff6347">getChannel()</span>-> pozovemo na **stream** neki

2. <span style="color:#ff6347">Files.neki()</span> -> ako otvorimo kanal moramo da definisemo i opcije nad njih pomocu klase <span style="color:#ff6347">StandardOpenOptions</span>

   #### StandardOpenOptions

   ovo dobijamo preko StandardOpenOptions.neki



## Callable

Ovo je genericki interfejs koji slicno radi kao `runnable`,njegov cilj je da izvrsi deo koda kao neki posao.Razlika u tome je sto ovaj vraca **Future**

Executors.neki<span style="color:#ff6347">.submit(Callable)</span>

## Executor

Ovo je interfejs koji izvrsava neki posao

<span style="color:#ff6347">Executor->ExecutorService->ScheduledExecutorService</span>

Kada napravimo instancu nekog Executor-a moramo da ga ugasimo kada u zavrsimo sa radom preko `shutdown`

<span style="color:#ff6347">Executors.neki</span> -> Ovako pravimo

## Atomic

Ovo su nove klase kao wraperi oko primitivnih tipova.Njih koristimo jer su <span style="color:#ff6347">threadsafe</span>



## Comparator

Ovo koristimo da uporedjujemo 

<span style="color:#ff6347">Comparator.comparing</span>(Person::getAge)

​					.thenComparing(...)

## Optional

Ovo je wrapper oko objekata da vidimo da li vrednost postoji.

<span style="color :#ff6347">Optional<T></span>

Optional.<span style="color:#ff6347">empty() </span>-> pravi prazan optional

​			.<span style="color:#ff6347">of(value)</span> -> pravi optional na osnovu vrednosti

​			,<span style="color:#ff6347">ofNullable(value)</span>

Kada imamo optional vrednosti mozemo pozvati metode:

<span style="color:#ff6347">get()</span>-> koja vraca objekat koji se nalazi u optionalu,ako se ne nalazi nista vrati null

<span style="color:#ff6347">isPresent()</span> ->koja vraca true ili false u zavisnosti da li postoji vrednost u optional objektu

<span style="color:#ff6347">isPresentOrElse(Supplier interface)</span> -> Ako nije present radi izvrsi suppliyer-a



## Date

Ovo je najstarija klasa za rad sa datumima,vecina metoda je depricated

```java
Date date=new Date();
Date date=new Date(longmilis);
Date date=Date.from(instant);
```

## Callendar

Novija klasa za rad sa datumima

Apstraktna klasa pravi se poomcu staticke metode

```java
Callendar c=Callendar.getInstance();
```

## GrigorianCallendar

```java
GrigorianCalendar gc=new GregorianCalendar(); ->give me curr date
    				new GregorianCallendar(year,month,days)
    				new GregorianCallendar(year,month,days,min,s)
    				new GregorianCallendar(Locale)
   					new GregoriaCallendar(TimeZone)
```

## TimeZone

Vremenska zona

```
TimeZone z =TimeZone.getDefault() -> get sys default
			TimeZone.getTimeZone(String) -> vidimo na netu stringove pa stavimo
			TimeZone.getTimeZone(ZoneId)
```

## Locale

Ovo je kao region locale

Srpski locale sa latinicom,bez ovoga Latn pisao bi cirilicu

```java
 new Locale("sr", "RS","Latn")
```

```java
Locale.neki
```

## Currency

```java
Currency.getInstance(Locale)
		.getAvailableCurrencies()
```

## Timer /TimerTask

Ove dve klase idu zajedno i sluze za organizovanje nekog rada koji treba da se izvrsi

TimerTask je apstraktna klasa,koja ima metodu `run`

<span style="color:#ff6347">schedule(timerTask,delay)</span>

```java
Timer t=new Timer();
t.schedule(TimerTask,Delay);
```

Napravimo neku klasu koja nasledjuje TimerTask i tu izvrsimo run



## Functional Interfaces

Ovo su interfejsi koji imaju samo 1 metodu.

Funkcionalan interfejs se oznacava sa @FunctionalInterface i ima samo 1 metodu.

<span style="color:#ff6347">Function<T, R> </span> -> prima objekat tipa `T` i vraca objekat tipa `R`

<span style="color:#ff6347">Predicate<T></span>  -> prima objekat tipa `T`i vraca `boolean`

<span style="color:#ff6347">Suplier<T> </span>-> `nema ulazne` argumente vraca obj tipa `T`

<span style="color:#ff6347">Consumer<T></span> -> prima objekat tipa `T` i ne vraca `nista`

<span style="color:#ff6347">UnaryOperator<T></span> -> podtip functinal,prima `T` i vraca `T` 

<span style="color:#ff6347">BinaryOperator<T></span> -> podtim functinal , prima `2 objekata tipa T` i vraca `T`

<span style="color:#ff6347">BiConsumer<T,R></span> -> podtim consumer-a,prima 2 ulaza tipa `T` i `R` ,i `ne vraca nista `

<span style="color:#ff6347">BiFunctional<T,U,R></span> ->prima 2 ulaza tipa `T` i `U` i vraca tip `R`

<span style="color:#ff6347">BiPredicate<T,U></span> -> prima  `T` i `U` i vraca  `boolean`

<span style="color:#ff6347">BooleanSupplier </span>-> suplier samo sa boolean kao return type

<span style="color:#ff6347">DoubleBinaryOperator</span> -> prima `2 double` ulaza i vraca `double`

<span style="color:#ff6347">DoubleConsumer </span>-> prima  `double` ,ne vraca `nista`

<span style="color:#ff6347">DoubleFunction<R></span> -> prima `double` ,vraca `R` 

<span style="color:#ff6347">DoublePredicate</span> -> prima `double`, vraca `boolean`

<span style="color:#ff6347">DoubleSupplier </span>-> `nema` ulaza, vraca `double`

<span style="color:#ff6347">DoubleToIntFunction</span> -> prima `double` vraca `int`

<span style="color:#ff6347">DoubleToLongFunction</span> -> prima `double` vraca `long`

<span style="color:#ff6347">IntBinaryOperator</span> -> prima `2 int` i vraca `int`

<span style="color:#ff6347">IntConsumer </span>-> prima `int` i ne vraca `nista`

<span style="color:#ff6347">IntFunction<R></span> ->prima `int` i ne vraca `R`

<span style="color:#ff6347">IntPredicate</span> -> prima `int` i vraca `boolean`	

<span style="color:#ff6347">IntSupplier </span>-> ne prima `nista `, vraca `int`

<span style="color:#ff6347">IntToDoubleFunction </span>-> prima `int` vraca `double`

<span style="color:#ff6347">IntToLongFunction</span> -> prima `int` vraca `long`





## Dates

### ChronoUnit

ChronoUnit.Nesto -> ovime imamo static pristup nekim metodama koje rade sa datumima

​							.between(Temporal1,Temporal2)

npr:

```java
ChronoUnit.DAY.between(localdate1,localdate2) -> daje nam razliku u danima izmedju ova dva
ChronoUnit.DAYS.between(instant1,instant2)
ChronoUnit.MONTHS.between(instant2,instant2)
                .
                .
                .
```

### Duration 

Ovo je iz paketa java.time

Duration.between(Temporal1,Temporal2) ->Ovo vraca `Duration` objekat pa mi iz njega mozemo da konvertujemo ovo u dane,mesece...

durationObject.toDays();

....

