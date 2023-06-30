# Spring MVC

Ovo je kao fullstack pravljenje aplikacije gde mi pravimo front i back u springu.

Koristimo neki template engine kao sto je <span style="color:tomato">ThimeLeaf</span>

On nam sluzi da dinamcki upravljamo i injectujemo kod u html.

Moramo da dodamo ThimeLeaf dependency za ovo

```xml
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

Kada radimo sa ovakim tipom aplikacija imamo kontroler,ali ne bas RestContoller vec samo Controller

<span style="color:tomato">@Controller</span>

Kada stavimo anotaciju controller on ce znati da se radi o MVC aplikaciji.Svrha tog kontrolera je da se iz svek funkcije vraca String,taj string je ime html dokumenta koji se prikazuje

```java

@Controller
public class UserController {

    private final UserServiceImpl service;


    @GetMapping()
    public String getIndex(){
        return "index";
    }

}
```

znaci da mi ovo index imamo koa html fajl.

Html fajlovi nam se nalaze u `resources/templates` i tu ce spring automatski da ih trazi



## Model

Ovo imamo uvek na raspolaganju i spring nam nudi ga a autiwire u svaku metodu u @Controller-u.

Sluzi nam da ubacujemo varijable dinamicki iz koda u html,i posle iz tog html samo izvlacimo kao key-value pair

```java
  @GetMapping("/new")
    public String getUserInsertPage(Model model){
        model.addAttribute("userCreationRequest",new UserCreationRequest());
        return "usersinsert";
    }
```

Znaci ovo addAttribute dodaje,i mi preko thymeleaf-a mozemo da izvucemo,ovo "userCreationRequest" je key za tu varijablu 

## Form

Kako radimo sa formama?

forma mora da posalje neku vrednost na beku,za sve to koristimo thymeleaf

prvo cemo imati stranicu koja ta forma vraca i prilikom vracanja te stranice mi moramo da inject prazan objekat koji pravimo da bi ga bind posle

```java
   @GetMapping("/new")
    public String getUserInsertPage(Model model){
        model.addAttribute("userCreationRequest",new UserCreationRequest());
        return "usersinsert";
    }

```

Ovde dodajemo prazan objekat koji ce se popuniti u formi,jer thymelaef ne zna da napravi ovaj objekat moramo mi da ga napravimo i da mu kazemo samo koji imput gde da mapira i sta da salje

Takodje moramo da kreiramo metodu koja ce da primi taj post request od forme i da doda usera

```java
   @PostMapping
    public String saveUser(@ModelAttribute("userCreationRequest") UserCreationRequest userCreationRequest){

        userService.insertUser(userCreationRequest);
        return "users";
    }
```

<span style="color:tomato">@ModelAttribute</span> ovo koristimo da bi mapirali objekat iz forme na nas,moraju imena da se poklapaju

Zadnji korak je sama forma

```java
  <form action="#" th:action="@{/users}" method="post" th:object="${userCreationRequest}">

      <div class="form-group">
        <label for="firstName">First Name</label>
        <input th:field="*{firstName}" type="text" class="form-control" id="firstName" aria-describedby="emailHelp" placeholder="Enter first name">
      </div>
      <div class="form-group">
        <label for="lastName">Last Name</label>
        <input  th:field="*{lastName}" type="text" class="form-control" id="lastName" aria-describedby="emailHelp" placeholder="Enter last name">
      </div>
      <div class="form-group">
        <label for="email">Email</label>
        <input th:field="*{email}" type="email" class="form-control" id="email" aria-describedby="emailHelp" placeholder="Enter email">
        <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input th:field="*{password}" type="password" class="form-control" id="password" placeholder="Password">
      </div>
      <div class="form-check mt-3 mb-3">
        <input th:field="*{enabled}" class="form-check-input" type="checkbox" value="" id="enabled">
        <label class="form-check-label" for="enabled">
          Enabled
        </label>
      </div>

      <button type="submit" class="btn btn-primary ">Save</button>
    </form>
```

<span style="color:tomato">th:action</span> -> na koji url ce da salje

<span style="color:tomato">th:object</span> -> objekat koji smo poslali da se mapira,on ce obaj obj da mapira

<span style="color:tomato">th:field="*{firstName}"</span> ovo stavljamo na input polja i tako znamo koje se polje mapira u koju varijablu

ako imamo checkbox onda ide malo drugacije

```java
<label>Profession:</label>
<select th:field="*{profession}">
    <option th:each="p : ${listProfession}" th:value="${p}" th:text="${p}" />
</select>
```

th:fiel stavljamo na select

## Redirect

MI redirect mozemo da radimo kada vracamo onaj string kao html template

```java
    @PostMapping
    public String saveUser(@ModelAttribute("userCreationRequest") UserCreationRequest userCreationRequest){

        userService.insertUser(userCreationRequest);
        return "redirect:/users";
    }
```

samo koristimo rec `redirect:putanja`

Mi mozemo da inject u svaku  @Controller klasu  <span style="color:tomato">RedirectAttributes</span>,ta klasa nam omogucava da set -ujemo parametre kada radimo redirect

```java
  @PostMapping
    public String saveUser(RedirectAttributes red){

        red.addFlashAttribute("message","nesto");
        return "redirect:/users";
    }
```

