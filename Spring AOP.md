# Spring AOP

AOP-> Aspect Oriented Programming

Ovo je module u spring framework-u,koji nam pomaze da definisemo reusable code koji se moze koristii u raznim slucajevima u aplikaciji.Ovo je znacajno u implementacijij cross-cutting concerns,kao sto su loggin,security,transaction managment.

U Spring AOP mi definisemo

-  <span style="color:red">Aspects</span> koji predstavljaju specificnu funkcionalnost,*<u>klasa u kojoj pisemo funkcionalnost</u>*
-  <span style="color:red">Pointcuts</span> koji definisu gde ce se aspekti primeniti
- <span style="color:red">Joint Point </span> ovo je point u app gde se aspekt moze prikljuciti
- <span style="color:red">Advice </span> ovo je kod koje se izvrsava(metoda) postoji 5 tipova
  - Before -> run advice(method) before target method execution
  - After -> run advice(method) after target method execution
  - After-Returning -> run advice(method) after method execution only if method completed successfully
  - After-Throwing ->run advice(method) after method execution only if method throws excetipon
  - Around -> run advice(method) before and after method execution

Na primer mi mozemo definisati logging aspect koji loguje pozive metoda ,ali pointcut ce odrediti gde ce se primeniti,koj scope.

Spring AOP koristi proxies da primeni aspekte na target objects.Kada se metoda pozove na target object AOP framework ce presresti poziv i primenice aspekat pre pozivanja svtarne metode.Ovo dozvoljava da mi separate concerns of app i da koristimo fleksibilno.

Sto nam je ovo bitno?

![image-20221218235927844](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221218235927844.png)

Vidimo da imamo 3 objekta  i hocemo da implementiramo log poruka,onda cemo morati da implementriamo log metodu na svakom od objekata kako bi ovo ispunili pa nije dobar dizajn.

Nacini da resimo?

1)

![image-20221219000027424](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221219000027424.png)

Ok izdvojicemo Logger.class koja ce da obavlja sve to

Jeidno sto ovo stvara problem je da dosta klasa zavisi od Logger-a sada,i imamo klasican kod  u svim metodama,moramo da zovemo te metode n puta ako imamo n objekata

2)

Uvodimo aspekte 

![image-20221219000555155](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221219000555155.png)

Aspekt nije bas klasa,mozemo je gledati kao klasa sa specijalnim sposobnostima.Kako je ovo drugacije od ovog nacina gore da imamo objekat kao,razlika je u tome sto mi necemo da referenciramo ovaj objekat u klasama,vec ce spring znati kako da ih poziva automatski,kako to zna pa preko configuration.

![image-20221219000710736](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20221219000710736.png)

Slicno kao triggers in DB

Pocinjemo dodavanjem dependency-ja

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



PointCuts: 

<span style="color:red">execution</span> -> ovaj pointcut match poziv metode

<span style="color:red">within</span> -> ovaj pointcut match odredjen type

<span style="color:red">this</span>-> ovaj pointcut match tacno jedan objekat koji smo napravili

<span style="color:red">@annotation</span> -> ovaj pointcut match anotacije na metodama koje smo stavili

Advice:

- <span style="color:red">@Before</span>
- <span style="color:red">@After</span>
- <span style="color:red">@Around</span>
- <span style="color:red">@AfterThrowing</span>
- <span style="color:red">@AfterReturning</span>

## Kako da definisemo Aspect?

Aspekt definisemo na nivou klase i tu klasu registrujemo kao bean neki kako bi je spring video i primenio

```java
@Aspect
@Slf4j
@Component
public class GeneralInterceptopAspect {
...
}
```

Ovde vidimo da je ova klase neki **Aspect** i da je **Bean**

## Kako da definisemo PointCut?

PointCut mozemo definisati na 2 nacina:

1)U posebnoj metodi,pa tu metodu da referenciramo u Advice

```java
@Aspect
@Slf4j
@Component
public class GeneralInterceptopAspect {
    
        @Pointcut("execution(* org.example.*.*(..))")
        public void logginPointCut(){
        }
    
      @Before("logginPointCut()")
    public void before(JoinPoint joinPoint){
        log.info("Beofre method invoked "+ joinPoint.getSignature());
    }
    
}
```

U definisanom Aspect-u napravimo **method Pointcut** i sada u Advice samo kazemo da koristi taj Pointcut.Ovako mozemo 1 pointcut koristiti za vise Advice-a ,ne moramo pisati opet.



2) Definisemo zajedno sa Advice-om.

```java
@Aspect
@Slf4j
@Component
public class GeneralInterceptopAspect {
    
   @After("execution(* org.example.*.*(..))")
    public void after(JoinPoint joinPoint){
        log.info("After method invoked "+ joinPoint.getSignature());
    }
    
}
```

Direktrno u Advice smo definisali Pointcut

Mi mozemo samo da napisemo  Advice(tj ovo @After itd) i da u njega ubacimo ovoo execuition tako ce on znati kako da izvrsi,ali ako to ne ubacimo moramo definisati @Pointcut i pokazati na njega kako bi on zna gde i kada da izvrsi.

Postoje vise vrsta pointcuta koje mi definisemo,ovde je execution

### execution

```java
* demo.shoppingcart.checkout()
* je returning type 
    
   execution(com.example.springaop.entity.Product com.example.springaop.controllers.MyController.getProduct())
    sada je return type product
    
   execution(com.example.springaop.entity.*com.example.springaop.controllers.MyController.getProduct())
    return type su sve klase iz paketa entity
    
    
    execution(void com.example.springaop.controllers.MyController.getProduct())
    return type je void

    
    execution(void com.example.springaop.controllers.MyController.get*())
    metoda pocinje sa get...
    
    execution(* com.example.springaop.controllers.MyController.getProduct(..))
    match ce metodu getProduct ali sa bilo kojim parametrima koje prima
```



### within

```java
  @Pointcut("within(* org.example.*.*(..))")
        public void logginPointCut(){
        }
```

mozemo definisati i within koji ce gledati u paketu

within match type,execution match method.Ovo nekad moze znaciti isto.

### annotation

```java
@Pointcut("@annotation(com.example.springaop.annotations.CustomAnotation)")
public void pointcutAnnotation(){

}
```

mormao napraviti custom anotaciju,i posaljemo put ka toj anotaciji,sve metode koje sadrze tu anotaciju koje se izvrse ce zadovoljiti ovaj pointcut

### this

```java
@Pointcut("this(com.example.springaop.controllers.MyController)")
public void pointcut(){

}
```

Ovo kaze da ce samo tu klasu da gleda,samo metode koje se izvrsavaju u njoj

## Kako da definisemo Advice?

Advice je ona metoda koja se u stvari izvrsava.Njega  definisemo sledecim koracima

### After

```java
    @After("pointcut()")
    public void after(JoinPoint joinPoint){
        log.info("After " + joinPoint.getSignature());
    }
```



### Before

```java
@Before("pointcut()")
    public void before(JoinPoint joinPoint){
        log.info("Before "+joinPoint.getSignature());
    }
```



### AfterReturning

```java
@AfterReturning(value = "pointcut()",returning = "product")
   public void afterReturning(JoinPoint joinPoint, Product product){
       log.info("After " + joinPoint.getSignature() + " returned "+ product);

   }
```

Ako radmo sa @AfterReturin imacemo returning value,ovde moramo paziti jer ako se izvrsi metoda koja ne vraca ovo a nas pattern ispunjava uslov,ovo u execution onda se nece advice uopset izvrsiti

fora je da kada definisemo i returning mi moramo da match sve u ovoj zagradi sto pise,metoda mora da zadovolji ovo execution i mora da ima ovaj returning ,ako pozovemo drugu metodu koja zadovoljava uslov ali ne vraca product onda nece logovati tj nece uci uopset ovde



### AfterThrowing

```java
    @AfterThrowing(value = "pointcut()",throwing = "productExceptioin")
    public void afterThrowing(JoinPoint joinPoint, ProductExceptioin productExceptioin){
       log.info("After " + joinPoint.getSignature() + " returned "+ productExceptioin.getMessage());

   }
```

Isto i sa throwing samo sto metoda mora da baca exception



### Around

```java
   @Around("pointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("Before "+joinPoint.getSignature());
        Object proceed = joinPoint.proceed();
        log.info("After "+joinPoint.getSignature());
       return proceed;
    }
```

Kada radimo sa around imamo nesto sto se zove ProceedingJoinPoint to imamo jer around objedinjuje After i Before 

pa moramo imati jasnu granicu kada ide jedno a kada drugo.Kada uradimo proceed posle toga je sve after i proceed vraca objekat metode koja ona vraca,ako ne vraca nita bice void ali ako vraca mozemo videti taj objekat



```java
@Pointcut("within(com.example.springaop.controllers.MyController)")
public void authentication(){}  

@Pointcut("within(com.example.springaop.controllers.MyController)")
public void authorization(){}

@Before("authentication() && authorization()")
public void auth(){
    
}
```



Mi mozemo kombinovati vise Pointcutova u jedan



```java
@After("pointcut()")
public void after(JoinPoint joinPoint){

    Product arg = (Product) joinPoint.getArgs()[0];
    log.info("After " + joinPoint.getSignature() + arg);
}
```

Takodje preko joinpointa moze izvuci arg ako metoda prima arg



