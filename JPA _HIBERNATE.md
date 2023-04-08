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

 

 

 

 

 

 

 

 

 

 

 

 

 

 

**@MapsId(value)**

Ovo value je za koje polje klase mapiramo

Ako je primarni ljuc child tabele isti kao primarni kljuc roditeljske tabele

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

 

 

**@Temporal(**

**TemporalType** :TemporalType -> ovo je tip koji se cuva (DATE,TIMESTAMP)

**)**

*Date Variable*

Ovo pisemo samo za date varijable

 

 

 

 

Multiple tables

 

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

 

**@SecondaryTable(**

**name** ->ovo je ime druge tabele

**,pkJoinColumns** -> ovo je kljuc po kojoj se te tabele join definisan dole

**)**

**@PrimaryKeyJoinColumn(**

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

 

 

**@JoinColumn(**

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
|      | ![img](file:///C:\Users\radoj\AppData\Local\Temp\msohtmlclip1\01\clip_image005.png) |

 



 

 

 

 

 

 

 

 

 

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

 