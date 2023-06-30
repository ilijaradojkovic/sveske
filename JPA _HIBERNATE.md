# JPA _HIBERNATE

## Class level annotations

<span style="color:#FA5F55">@Entity</span>**(**

​	**Name** :String ->Daje ime tabeli

**)**

```java
@Entity(...)
class ....
```

Ovo je obavezna anotacija kako bi spring data jpa prepoznao klasu kao tabelu,kako bi orm prepoznao 





 <span style="color:#FA5F55">@Table</span>**(**

​	**Name** :String -> ovo je ime tabele,mozemo i u entity da koristimo isto je

​	**Catalog** :String ->

​	**Schema** :String

​	**UniqueConstraints** : UniqueConstraints[]

​	**Indexes** : Index[]

**)**

```java
@Table(...)
class ....
```

Ovo se koristi ako zelimo da specificiramo vise informacija o tabeli

 

## Variable level annotation

<span style="color:#FA5F55">@Column</span>**(**

​	**Name** :String -> Definisemo ime kolone

​	**Unique** :Boolean -> Definisemo da li je kolona unique

​	**Nullable** :Boolean -> Definisemo da li je kolona nullable

​	**Insertable** :Boolean -> Definisemo da se ova kolona moze ubacivati u tabelu,da li je insert dozvoljen

​	**Updatable** :Boolean -> Definisemo da li je update dozvoljen nad ovom kolonom

​	**ColumnDefinition** :String -> Ovo je string reprezentacija tipa podataka kolone

​	**Table** :String ->kojoj tabeli pripada

​	**Lengt** :int->duzina

​	**Precision** :int->pokretni zarez

​	**Scale** :int 

**)**

```java
@Column(...)
typeVariable Variable
```





<span style="color:#FA5F55">@Id</span>

```java
@Id(...)
typeVariable Variable
```

Ovo je primarni kljuc tabele

 



<span style="color:#FA5F55">@GeneratedValue</span> **(**

​	**Strategy**  :GenerationType -> ovo je tip tj kako generise vrednost

​	**Generator** :String

)

 GenerationType.AUTO -> Ovo je da mi damo Hibernatu da odluci kako ce

GenerationType.IDENTITY ->Ovo je jednako auto-increment u bazi

GenerationType.SEQUENCE -> Ovo samo ako baza podrzava,moramo definisati generator

```java
@Id 

@GeneratedValue(generator = "sequence-generator")

@GenericGenerator( 

 name = "sequence-generator",

 strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",

 parameters = { 

@Parameter(name = "sequence_name", value = "user_sequence"),

@Parameter(name = "initial_value", value = "4"), 

@Parameter(name = "increment_size", value = "1") 

} ) 

private long userId;
```

 

<span style="color:#FA5F55">@Version </span>



<span style="color:#FA5F55">@OrderBy</span> **(**

​	**Value** :String ->Ovde stavljamo string izraz za sortiranje

**)**

```java
@OrderBy(“id asc”) ->on po default vec poredja po rastucem ovde koristimo jpql
Private List<Employee> employees;
```



<span style="color:#FA5F55">@Transient</span>

```java
@Transient
Variable
```

Nece da persist ovaj podatak

 



<span style="color:#FA5F55">@Lob</span>

```java
@Lob
private byte[] image;
```

Ovo je za blob objekte

 



Pored prostih kljuceva,hibernate nam omogucava da kreiramo komplikovan primarni kljuc.Ovo mozemo postici kroz nekoliko primera

<span style="color:#FA5F55">@EmbeddedId i @Embeddable</span>

Ako imamo dva ili vise polja koji su pk.

```java
@Embeddable

Public class OrderEntryPK implements Serializable{

​        Private long orderId;

​        Private long productId;

}

@Entity

Public class OrderEntity{

@EmbeddedId

Private OredrEntryPk id;

}
```

For a je da embeddable bude klasa koje ce da sadrzi vise polja,ta klasa mora da implementuje Serializable,

A druga klasa tj entity koji prezistujemo mora da bude oznacen sa EmbeddedId

 

<span style="color:#FA5F55">@IdClass</span>

 Ovo je slicno kao EmbeddedId samo sto kljuceve ovde direktno definisemo u entity klasi.Svaki pk je @Id svoj 

```java
@Entity

@IdClass(OrderEntryPk.class)

Public class OrderEntry{

@Id

Private long orerId;

@Id

Private long productId;

}

Public class OrderEntryPK implements Serializable{

​        Private long orderId;

​        Private long productId;

}
```

 

Imena varijabli moraju biti ista

 

<span style="color:#FA5F55"> @Version</span>

Kazemo koja je verzija entiteta u bazi,ovo omogucava `optimistic locking` . Kada se perzistentan objekat update polje koje je oznaceno sa **@Version** ce se promeniti jer se entity promenio.Pa kada se taj novi entity update db ce da proveri verziju polja da nema neka transakcija koja ga drzi,time omogucavamo monu sinhronizaciju tj onaj lock .

 On ce da update data samo ako su verzije iste,ako je neko promenio  i broj je drugi,a ta transakcija ima broj 1 dok je u bazi broj 2 onda nece da update

 

```
@Entity
public class Product {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
 
    @Version
    private Integer version;
 
    private String name;
    private Double price;
 
    // constructors, getters, setters, etc.
}
```

 

 

 

 

 

 

 

 

 

 

<span style="color:#FA5F55"> @MapsId(value)</span>

Ovo value je za koje polje klase mapiramo

Ako je primarni ljuc child tabele isti kao primarni kljuc roditeljske tabele

```java
@Entity
Public class User{
    Private String email;
    Privat String password;
}

@Entity
Public class UserDetails{
    @Id
    Private int id;

    @MapsId(value=”email”)
    @OneToOne
    Private User user;
}
```

 

 

<span style="color:#FA5F55"> @Temporal</span>**(**

**TemporalType** :TemporalType -> ovo je tip koji se cuva (DATE,TIMESTAMP)

**)**

*Date Variable*

Ovo pisemo samo za date varijable

 

 

 

 

## Multiple tables

 

![img](file:///C:\Users\radoj\AppData\Local\Temp\msohtmlclip1\01\clip_image001.png)Mi mozemo da mapiramo 1 klasu (1 entity) na vise tabela



|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\radoj\AppData\Local\Temp\msohtmlclip1\01\clip_image002.png) |

 



 

 

 

 

 

 

 

 

 

U Employee entity se mapiraju dve tabele,koje nemaju smisla da stoje odvojeno nego ih ovako stavimo u jednu klasu.

| **Employee_Details** |
| -------------------- |
| **Details_id**       |
| **Department**       |
| **sex**              |

| **Employee** |
| ------------ |
| **Id**       |
| **Salary**   |
| **Name**     |
|              |

 



|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\radoj\AppData\Local\Temp\msohtmlclip1\01\clip_image003.png) |

 



 

 

Details_id ima fk ka id.

```
@Entity
@Table(name = "Employee")
@SecondaryTable(name = "Employee_Details",pkJoinColumns = @PrimaryKeyJoinColumn(name = "details_id",referencedColumnName = "id"))
public class Employee {

    @Id
    private Integer id;


    private String name;

    private Integer salary;
    @Column(name = "department",table = "Employee_Details")
    private String department;
    @Column(name = "sex",table = "Employee_Details")
    private String sex;
}
```

 

 

Glavne anotacije

 

<span style="color:#FA5F55"> @SecondaryTable</span>**(**

**name** ->ovo je ime druge tabele

**,pkJoinColumns** -> ovo je kljuc po kojoj se te tabele join definisan dole

**)**

<span style="color:#FA5F55"> @PrimaryKeyJoinColumn</span>**(**

**name,** -> ovo je ime kolone u novoj tabeli,njen pk

**referenceColumnName** -> ovo je referencirana kolona u nasem entitetu

**)**

**
**

 

## Multiple tables with foreign keys

 

| **Address** |
| ----------- |
| **id**      |
| **city**    |
| **country** |

| **Employee**   |
| -------------- |
| **Id**         |
| **Salary**     |
| **Name**       |
| **Address_id** |

 



|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\radoj\AppData\Local\Temp\msohtmlclip1\01\clip_image004.png) |

 



 

 

U ovom slucaju imamo secondary table koja je referencirana od strane employee tabele preklo address_id kolone.

 

 

<span style="color:#FA5F55"> @JoinColumn</span>**(**

**Name,** ->ovo je ime kolone u tabeli

**referenceColumn,** -> ovo je referencirana kolona u toj drugoj tabeli sa kojom povezujemo

**unique,** 

**nullable,**

**insertable**

**,updatable,**

**columnDefinition,**

**table,**

**foreignKey**

**)**

**
**

 

# Table inheritance

 



|      |                                                              |
| ---- | ------------------------------------------------------------ |
|      | ![img](file:///C:\Users\ilija\AppData\Local\Temp\msohtmlclip1\01\clip_image005.png) |

 



 

 

 

 

 

 

 

 

 

Nekad cemo imati ovakvu situaciju u nasem kodu,mi ovo moramo da mapiramo u bazu.Postoje 2 nacina

1)Napravimo 1 tabelu za sve i sve informacije i samo na osnovu tipa ih razlikujemo

2)Napravimo pojedinacne tabele za SmallProject i LargeProject i jasno ih odvojimo

 

U zavisno sti sta nama treba mozemo da koristimo neka od ova dva principa

 

Polazi od nacina da u bazi hocemo da pamtimo samo project i da nekako ove tabele integrisemo u tu jednu automatski preko anotacija.

 

Vazne anotacije za ovo su :

**@Inheritance(strategy)** -> ovo odredjuje kako ce jpa da gleda i mapira ovo

**@Discriminator(name)** -> ovo kaze koje ce polje u tabeli da bude tip koji odredjuje da li je large project ili small project.

**@DiscriminatorValue(name)** -> ovo stavljamo na konkretnim kalsama (Specijalizacije apstrakcije) i ta vrednost ce biti upisana u polje tabele( koje je definisano u @Discriminator-u)

**@MappedSupperclass**  -> Ova anotacija se stavlja nad superklasom koja je nasledjena od strane specijalizacija,samo ce se specijalizacije sacuvati kao tebele i povuci sve vrednosti iz superklase,dok se nece cuvati superklasa

 

 

## Single Table Inheritance

Single Table jer se cuva samo 1 tabela,koja ima info o svima

```
@Entity
@Inheritance(strategy=InheritanceType.JOINED)
@DiscriminatorColumn(name="PROJ_TYPE")
@Table(name="PROJECT")
public abstract class Project {
  @Id
  private long id;
  
  private String name;
  ...
}
@Entity
@DiscriminatorValue("L")
public class LargeProject extends Project {
  private BigDecimal budget;
}
 
@Entity
@DiscriminatorValue("S")
public class SmallProject extends Project {
}
 
```

 

Napravice se tabela koja ce da sadrzi sve vrednosti i LargeProject i SmallProject ii mace kolonu novu PROJ_TYPE u kojoj ce da budu vrednosti tipa klase koja je iz DiscriminatorValue

 

 



 

## Joined, Multiple Table Inheritance

Ovde se cuvaju vise tabela ,tj svaka posebno,i u glavnoj samo tip 

 

```
Entity
@Inheritance(strategy=InheritanceType.JOINED)
@DiscriminatorColumn(name="PROJ_TYPE")
@Table(name="PROJECT")
public abstract class Project {
  @Id
  private long id;
  
  private String name;
  ...
}
@Entity
@DiscriminatorValue("L")
@Table(name="LARGEPROJECT")
public class LargeProject extends Project {
  private BigDecimal budget;
}
@Entity
@DiscriminatorValue("S")
@Table(name="SMALLPROJECT")
public class SmallProject extends Project {
}
```

 

 



 

## Table Per Class Inheritance

Samo ce se napraviti konkretne klase ,te tabele samo,ova nadklasa ce da se ignorise

 

```
@Entity
@Inheritance(strategy=InheritanceType.TABLE_PER_CLASS)
public abstract class Project {
  @Id
  private long id;
  ...
}
@Entity
@Table(name="LARGEPROJECT")
public class LargeProject extends Project {
  private BigDecimal budget;
}
@Entity
@Table(name="SMALLPROJECT")
public class SmallProject extends Project {
}
```

 

 

Isto ovo radi i 

 

```
@MappedSuperclass
public abstract class Project {
  @Id
  private long id;
  @Column(name="NAME")
  private String name;
  ...
}
@Entity
@Table(name="LARGEPROJECT")
public class LargeProject extends Project {
  private BigDecimal budget;
}
@Entity
@Table("SMALLPROJECT")
public class SmallProject extends Project {
}
```

##  Relationships

### Cascade

Ovo je osobina sta se radi sa objekom koji je nestovan(inner) u nekmo.Npr imamo User i Address, user sadrzi objeat address

onda je on inner ili se kaze da user "poseduje" address.Ova osobina nam omogucava da kada se izvrsi npr insert nad userom da se automatski insert i taj address objekat



### Fetch 

Ovo radimo preko enuma `FetchType` 

Ova osobina koju dodeljujemo je da li ce se svi podaci izvuci iz baze odmah ili ce se uraditi posle kada nam zatreba.

Npr ovde imamo User i Address,user sadrzi objekat adress i kaze se da on sadrzi address.

Postoje 2 tipa FetchType-a:

1. Eager -> Dovicu mi ceo objekat odmah
2. Lazy -> Dovuci mi objekat kasnije kada mi zatreba



### @OneToOne

![image-20230626161725472](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230626161725472.png)

OneToOne ce da mapira jednu klasu za drugu gde imamo **samo** 1 pojavljivanje ove druge

TO znaci da ce user imati samo 1 adresu,ta adresa postoji samo kao jedna i niko drugi ne moze da se poveze na nju niti da ima duplikata,bukvalno samo 1 adresa moze da posotji i da se veze za to user-a.



```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;

    //...

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    @PrimaryKeyJoinColumn
    private Address address;

    //... getters and setters
}

@Entity
@Table(name = "address")
public class Address {

    @Id
    @Column(name = "user_id")
    private Long id;

    //...

    @OneToOne
    @MapsId
    @JoinColumn(name = "user_id")
    private User user;
   
    //... getters and setters
}
```

<span style="color:#FA5F55">@OneToOne(</span>

cascade -> CascadeType,

fetch -> FetchType,

optional -> boolean,

orphanRemoval -> boolean

<span style="color:#FA5F55">)</span>

### @ManyToOne() i @OneToMany()

Ovo u sustini stvara tu novu tabelu jer sse sadrzi vise vrednosti.Imaecmo klase **A** i **B**(B ce da sadrzi vise objekata tipa A)

ManyToOne -> se stavlja na entity objekat **A** koji ima samo 1 objekat  tipa **B**,  dok ovaj  tip **B** sadrzi vise tih objekata **A **

OneToMany -> se stavlja na entity objekata **B** koji ima vise objekata tipa A.

<span style="color:#FA5F55">@ManyToOne(</span>

cascade, -> CascadeType

fetch, -> FetchType

optional

<span style="color:#FA5F55">)</span>

<span style="color:#FA5F55">@OneToMany(</span>

cascade -> CascadeType

fetch , -> FetchType,

orphanRemoval, -> Boolean

mappedBy -> String

<span style="color:#FA5F55">)</span>



```java
@Entity(name = "roles")
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @OneToMany
    private List<User> userList;
}


@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "email",length = 128,nullable = false,unique = true)
    private String email;

    @Column(name = "enabled")
    private Boolean enabled;

    @ManyToOne()
    private  Role role;
}
```

Ovde cese napraviti nova tabela  jer role sadrzi vise user-a nova tabela ce biti kreirana.Tako da ce se kreirati tabela User, Roles,i ta nova tabela koja je kao agregacija te dve roles_user_list koja sadrzi samo id role i id user-a.

Kada se kreira ta nova tabela njoj mozemo da upravljamo pomocu anotacije 

### @JoinTable

<span style="color:#FA5F55">@JoinTable(</span>

name ,

catalog,

schema,

joinColumns -> JoinColumn[], -> u tamo klasi sa kojom pravimo vezu id 

inverseJoinColumns -> JoinColumn[], -> u trenutnoj klasi id

foreignKey, -> ForeignKey,

inverseForeignKey,ForeignKey,

indexes-> Index[],

iniquieConstraints -> UniqueConstaint[]

<span style="color:#FA5F55">)</span>

### @JoinColumn

<span style="color:#FA5F55">@JoinTable(</span>

name,

referenceColumnName,

unique,

nullable,

insertable,

updatable,

columnDefinition,

table,

foreignKey-> ForeignKey

<span style="color:#FA5F55">)</span>

```
@Entity
@Table(name = "parent")
public class Parent {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany
    @JoinTable(
        name = "parent_child",
        joinColumns = @JoinColumn(name = "parent_id"),
        inverseJoinColumns = @JoinColumn(name = "child_id")
    )
    private List<Child> children;

    // Getter and Setter methods
}

@Entity
@Table(name = "child")
public class Child {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Other properties

    // Getter and Setter methods
}
```

### @ManyToMany

Ovo je kada imamo vise-vise vezu

<span style="color:#FA5F55">@ManyToMany(</span>

cascade,

fetch

mappedBy

<span style="color:#FA5F55">)</span>

Uz ovu  anotaciju moramo da stavimo anotaciju @JoinColumn kako bi on znao gde sta da mapira

```java
@Entity(name = "roles")
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(length =40,nullable = false,unique = true)
    private String name;

    @Column
    private String description;

    @ManyToMany()
    private List<User> userList;
}

@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "email",length = 128,nullable = false,unique = true)
    private String email;

    @ManyToMany()
    @JoinTable(
            name = "users_roles",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles= new HashSet<>();

}
```

kako Role.class ima ovo @ManyToMany i User.class ima @ManyToMany ali takodje smo stavili i @JoinTable on ce kreirati 4 tabele

User

Role

@ManyToMany -> default tabela

@JoinTable -> tabela sto smo tu definisali



Mi hocemo samo 3,ovo default mozemo da izbacimo tako sto ubacimo mappedBy=""

mappedBy stavljamo u onoj klasi gde nemamo @JoinTable i to oznacava ko ima "vlasnistvo",ko je glavni u toj vezi,mappedBy="ime varijable u drugoj klasi koja drzi tu ManyToMany vezu"

```
@Entity(name = "roles")
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(length =40,nullable = false,unique = true)
    private String name;

    @Column
    private String description;

    @ManyToMany(mappedBy = "roles")
    private List<User> userList;
}

@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "email",length = 128,nullable = false,unique = true)
    private String email;

    @ManyToMany()
    @JoinTable(
            name = "users_roles",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles= new HashSet<>();

}
```

