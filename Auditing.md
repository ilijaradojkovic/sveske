# Auditing

Tracking and loggin changes 

Pratimo promene tabela

Kljucne anotacije:

1. <span style=color:#DC4C4C>@PrePersist</span>
2. <span style=color:#DC4C4C>@PreUpdate</span>
3. <span style=color:#DC4C4C>@PreRemove</span>

Ovo postavljamo na entity klasama kako bi imale ovaj event da ih spring pokupi i registruje kao auditing.

Ako imamo vise klasa koje obradjuju ovo mozemo definisati jednu klasu koja sve ovo obradjuje 

```java
@EntityListener(MyClass.class)

entityClass{

}
class MyClass{
@PrePersist
@PreUpdate
@PreRemove
 public void Handle(Object object){
     ...
 }
}
```

Ove anotacije stavljamo na metode,i ovo je dosta primitivno postoje bolji nacini.

Jpa nema po defalt-u ovo audited pa moramo dodati dependency **hibernate_envers**

tada mozemo koristiti oznake 

- <span style=color:#DC4C4C>@Audited</span>
-  <span style=color:#DC4C4C>@NotAudited</span>

```java
@Entity
@Audited
class A{
...
}
```

Automatski ce dodati novu tabelu za auditing



```java
@Entity
@Audited
class A{
...
@OneToOne
@Audited ILI @NotAudited
B b;
}

@Entity
(@Audited/@NotAudited ili ovde da oznacimo)
class B{

}
```



Klasa A ako ima neko mapiranje mora se oznaciti ili da je taj objekat @Audited ili @NotAudited

To mozemo uraditi ili na varijabli ili na samoj klasi B

Postoji jos jedan nacin da ovo uradimo,spring data jpa dolazi uz neki auditing pa mozemo njega ukljucit

Moramo provo to omoguciti 

<span style=color:#DC4C4C>@EnableJpaAuditing</span> -> ovo omogucava da koristimo auditing

<span style=color:#DC4C4C>@CreatedBy</span>

<span style=color:#DC4C4C>@CretedDate</span>

<span style=color:#DC4C4C>@LastModifiedBy</span>

<span style=color:#DC4C4C>@LastModifiedDate</span>

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @CreatedBy
    private String createdBy;

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedBy
    private String lastModifiedBy;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}
```

