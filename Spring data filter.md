#  Spring data filter

Bitne anotacije koje su kljucne:

- @FilterDef ->Daje definiciju filtera,ne mora biti na klasi samo moze i na paketu.
- @Filter ->Ovo povezuje entity klasu sa filter definicijom

<span style="color:red">@FilterDef(name,defaultCondition,parameters)</span>



 	name-> ime filtera

​	defaultCondition -> sta filter izvrsava

​	parameters -> koji su parametri nad kojima izrsava



<span style="color:red">@Filter(name,condiiton)</span>

```java
@FilterDef(name="activeFilter",defaultCondition = "active = :active", parameters={@ParamDef(name="active", type="boolean")})
@Filter(name ="activeFilter")
public class MUser {

    @Id
    private UUID id;
    private String name;
    private boolean active;
}
```

Definisemo filter i povezemo ga sa ovom entity klasom,ovime on jos nije enabled vec samo povezan.

Da ga aktiviramo koristimo **EntityManager**

```java
Session unwrap = entityManager.unwrap(Session.class);
Filter activeFilter = unwrap.enableFilter("activeFilter");
activeFilter.setParameter("active",true);
List<MUser> select_m_from_mUser_m = unwrap.createQuery("select m from MUser m", MUser.class).getResultList();
System.out.println(select_m_from_mUser_m);
```

Ovo su bili filteri koji imaju default condition,imacemo vise filtera koji se povezuju na definiciju koja ima default condition,ako hocemo dinamici condition to onde:

```java
@FilterDef(name="activeFilter", parameters={@ParamDef(name="active", type="boolean")})
@Filter(name ="activeFilter",condition="...")
public class MUser {

    @Id
    private UUID id;
    private String name;
    private boolean active;
}
```

Sklonimo default condition sa @FilterDef i ubacimo @Filter(condition)

sa @Filter mozemo oznaciti i kolone



### Kako ovo da koristimo sa JPA?

isto sve ,stavimo filterdef na MUser-a i FIlter ,a jer repository radi sa klasom tog tima on ce da primeni filter nad tim entitetmo,samo pre poziva repository moramo 

```java
Session unwrap = entityManager.unwrap(Session.class);
unwrap.enableFilter("activeFilter");
List<MUser> test = repo.findByName("test");
System.out.println(test);
```