# Angular

## Dodati Bootstrap

Instaliramo bootstrap u angular projektu preko 

```cmd
npm install bootstrap jquery --save
```

sada imamo foldere u node_modules

```js
node_modules/jquery/dist/jquery.min.js
node_modules/bootstrap/dist/css/bootstrap.min.css
node_modules/bootstrap/dist/js/bootstrap.min.js
```

da bi radio mormao da dodamo,tj da referenciramo ove fajlove u `angular.json`

```js
"styles": [
    "styles.css",
    "./node_modules/bootstrap/dist/css/bootstrap.css"
  ],
  "scripts": [
    "./node_modules/jquery/dist/jquery.min.js",
    "./node_modules/bootstrap/dist/js/bootstrap.min.js"
  ],
```

## Components

Komponente su u stvari UI deo koji je reusable.

Mi definisemo komponente  i reuse ih na vise HTML stranica



![image-20230629095748719](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230629095748719.png)

Za svaku komponentu definisemo klase koje ce njima upravljati,tu ce se nalaziti kod koji ce menjati stanja UI dela tj komponenti.

### Kako deklarisemo komponente?

Da bi nesto ucinili komponentom moramo da imamo klasu koja je vezana za nju(komponentu) i da ta klasa sadrzi `dekorator` (anotaciju kao da je komponenta).

Proces se sastoji iz 2 koraka:

1. Kreiraj klasu
2. Kreiraj dekorator

Mada se sve ovo automatski radi kada napravimo pomocu frameworka

```
@Component({
  selector:
  templateUrl: 
  styleUrls:
})
export class AppComponent {

}

```

<span style="color:tomato">selector</span> -> Ovo je kako ce se zvati komponenta,kako cemo je zvati u HTML-u.Naziv komponente u HTML-u

<span style="color:tomato">templateUrl</span> ->ovo je url gde nam se nalazi izled komponente,HTML fajl

<span style="color:tomato">template</span> -> ovo je isto kao templateUrl samo sto se ovde odmah gleda fajl u komponenti,a sa ovim url mozemo da gledmao i izvan

<span style="color:tomato">styles</span> -> ovo je isto kao stylesUrls samo sto gleda u kompoentnti 

<span style="color:tomato">styleUrls</span> -> ovo  je niz gde se nalaze css fajlovi za html

npr:

```
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],

})
export class AppComponent {
  title = 'angularTest';
}

```

Komponenta nam se zove "app-root" i tako cemo je koristiti u html-u ,pomocu tog naziva.To je sada naziv taga

<app-root></app-root>

### Binding field in html

Ovo je nacin da mi **bind** polja u html-u,i postoje 4 nacina:

1. One-way binding (interpolation)

   Interpolation je pretvasranje u string,bukvalno uzmemo varijablu i pretvorimo je u string i to je to.Nema nekog bindovanja da reaguje na promene vec samo u string pretori.

   To se radi pomocu : <span style="color:tomato">{{ime varijable}}</span>

   

2. Property binding

   Ovo bind radi sa vise konkretnih tipova 

   boolean->boolean, a  interpolation bi uradio sa boolean->string

   To se radi pomocu: <span style="color:tomato"> [property tag-a]="varijabla"</span>

   Zato se zove proeprty jer mi bind varijablu za neki property taga

   npr:

   ```
   <img [src]="urlImage">
   [style.width.px]="nesto" moze i ovako
   ```

   

3. Two-way binding

   ​	Najpoznatiji nacin da bind,omogucava nam da bind varijablu klase tako da se promene odraze na UI delu.One-way je samo ako se promeni u kodu odrazice se na UI delu.

   To se radi pomocu: <span style="color:tomato">[(ngModel)]="imePolja"</span>  mora da ima **name** u input delu

   ```
   <input type="text" [(ngModel)]="searchText">
   ```

   Dodatno moramo da imamo get i set metode za ovo polje u .ts 

   ```
   export class myCompoennt{
   	searchText:string="";
   	get searchText():string{return this.searchText;}
   	set searchText(value:string){this.searchText=value;}
   }
   ```

   nisu nam potrebne get i set metode

4. Event binding

   ​	bind vec postojace event-e za funkcije klase kompoenente

   ​	To se radi pomocu:

   <span style="color:tomato">(ime event-a)="ime funkcije"</span>

   npr:

   ```
   (click)="handleClick()"
   ```

   i onda u .ts

   ```
   export class MyComponent{
    handleClick(){
    	..
    }
   }
   ```

   

Svaku komponentu koju napravimo moramo da stavimo u `modules`,ali ovo se autmatski radi kada imamo framework



<span style="color:tomato">@NgModule({</span>

​	<span style="color:tomato">declarations</span> -> stavljamo komponente da ih register

<span style="color:tomato">	imports</span>-> register other modules

<span style="color:tomato">	providers</span>->

​	<span style="color:tomato">bootstrap</span> ->

<span style="color:tomato">})</span>

## Directives

Delimo ih na 

1. Attribute Directives -> menjaju izgeld elementa
2. Structural Directices -> Dodaju elemente

BIlo koji directive se radi sa [imedirective],

structural directives imaju *  jedini

Operatori u HTML-u.

Mi imamo mogucnost da pisemo neke operatore,funkcije u html-u kao kada smo raidli sa thymeleaf-om

<span style="color:tomato">*ngIme</span> ovo je opsta sintaksa

### if

<span style="color:tomato">*ngif=ifLogic</span>

​	Prikazuje ili sakriva komponente u zavisnosti da li je true ili false



```html
<p *ngIf="display; else customtemplate"> cao</p>
<custom-template #customtemplate>
    <p> display in else</p>
</custom-template>
```

<span style="color:tomato">*ngfor=forlogic</span>

```
	*ngfor="let p of products"
```

### switch

<span style="color:tomato">*[ngswitch] </span>

```
<div [ngswitch]="variable">
	<span *ngSwitchCase =value
		  *ngSwitchDefault
```

### style

<span style="color:tomato">[ngStyle] </span> 

Ovo nam sluzi da dinamicki odredjujemo stil ako je neki uslov ispunjen

```
[ngStyle]={color: p.available=== 'Available'? 'Green':'Red'} 
```

### class

<span style="color:tomato">[ngClass]</span>

Sluzi nam da dodamo dinamicki klasu 

```
[ngClass]="{className: searchValue!=''}"
```

<span style="color:tomato"><ngContent></span> dinamicki ubacimo kontent HTML

Ako imamo definisanu komponentu

```
<mycomponent>
	...
</mycomponent>
```

a struktura mycomponent html-a

```html
<div>

<p></p>
<div>
```

e sada ovo sto smo ubacili gore sa ... nece biti prikazano u samoj kompoenenti mycompoenent,jer u tom html nema nista da kao ubaci sta je u ovim ... za to koristimo <ng-content></ng-content>

mi pass content u component selector-u

Mi nismo ograniceni da imamo 1 ng-content vec mozemo vise  samo ih nekako mozemo oznaciti da bi znao kada ide koj .

To radimo pomocu `select` opcije

<ng-content select='[tagcss]'></ng-content>

i onda  u deci stavimo 

```
<my-component>
	<p tagcss ></p>
</my-component>
```

<ng-container>

Ovo je samo container koji provajduje angular on ne dodaje nikakav stil,vec je samo wrapper i na njega primenjujemo druge direktive npr *ngif

### custom

Hocemo da kreiramo custom attribute koji menja boju u zelenu

Napravimo fajl `ime.directive.ts`

```ts
@Directive({
	selector: '[setBackground]'
})
export class SetBackgroundDirective{
	
	constructor(element:ElementRef){
		element.nativeElement.style.background="#GEW218";
	}
}
```

Ako ovu directive koristimo na bilo koj elem on ce u konstnruktoru da je prosledi,takodje da ovo koristimo moramo da ga ubacimo u module,ovo se automatski radi kada imamo framework.

i sada samo ubacimo ovaj directive u html tag

```html
<p setBackground>cao</p>
```

### custom structural directive

`Template<any>` -> sta dodajemo i brisemo

`ViewContainerRef `-> conateiner gdese to nalazi



```ts
@Directive({
	selector: '[appCustomDirectiveIF]'
})
export class SetBackgroundDirective{
	
	constructor(private template:Template<any>,private viewContainer:ViewContainerRef){}
    
    @Input() set displayView(condition:boolean){
        if(condition){
            this.viewContainer.createEmbeddedView(this.template);
        }else{
            this.viewContainer.clear();
        }
    }
}
```

Mormao da mu stavimo ime directive i onda input variable,jer ne bi znao gde da gleda ,pre kada smo pravili one attribute custom directive nismo morali da stavljamo nista vec samo ime ,ali ovde ne ovde ide druacije

```html
<div *appCustomDirectiveIF "displayView='nekiboolean'">
```

znaci ako je ovaj boolean tacan display ce se ,a  u suprotnom nece

## Angular Lifecycle

Angular prolazi kroz razne faze tokom rada,mi te interfejse moramo u compoennnt da bi overide taj lifecycle i da ga koristimo.

Angular lifecycle prvo render root komponentu pa onda tek decu.

### Change detection

Ovo je mehanizam koji angular ima,i sinhronizuje komponente.

npr imamo <div>Hello {{name}}</div> ,angular ce da update DOM svaki put kada se name varijabla promeni

<span style="color:tomato">onInit</span>

<span style="color:tomato">onChanges</span>

<span style="color:tomato">onDestory</span>

<span style="color:tomato">AfterViewInit</span>

## Pipes

Pipes je nacin na koji se data transofrmise.

Radimo formatiranje podataka.

Da bi definisali pipie moramo da korsitimo dekorator

<span style="color:tomato">@Pipe(...)</span>

```
@Pipe({
	name:"ime"
})
export class MyCustomPipe implements PipeTransform
{
	transform(value:string):string{
		...
	}
}
```

Moramo da implementiramo angular interface <span style="color:tomato">PipeTransform</span> koji nam nudi metodu transform i onda nju koristimo da transformisemo podatke.

Da bi priomenili taj pipe,ili bilo koji koristimo <span style="color:tomato">| ime pipe</span> operator

```
<h1>{{Title|imepiepe}}</h1>
```

Mozmeo da kombinuje i vise njih

```
<h1>{{Title|imepiepe| pipe2|pipe3}}</h1>
```

Takodje postoje i default pipovi kosu su vec definisani u frameworku

#### Passing param into pipe

```
@Pipe({
	name:"ime"
})
export class MyCustomPipe implements PipeTransform
{
	transform(value:string,params):string{
		...
	}
}
```

i sada kada pisemo moramo da ubacimo taj param

```
<h1>{{Title|imepiepe: imevarijable}}</h1>
```

#### Pure Pipe

Samo namestimo kada ga definisemo za property,kada se promeni referenca to je pure change

```ts
@Pipe({
	name:"ime",
	pure: false/true
})

```

Pure pipe znaci da je primitivan input(int string...) u pipe

#### Impure Pipe

ako radimo sa objektima moramo da zamenimo ceo objekat ,npr ako raidmo sa nizovima ne mozemo samo dadodamo vec ceo array da zamenimo

u zavisnosti kako se vrednost menja odlucuje da li ce se pipe pozvati,ako je namesten na pure pipe ,pozvace se samo ako se cela referenca promeni.

#### Async Pipe

<span style="color:tomato">| async</span> 

ovo je vec build-in pipe koji vec posotji u angularu,njega stavljamo ako imamo promice,osbervalble i zelimo da se taj input kada je dosputan da se display ,ako stavimo samo 

{{variable_async}} nece da display resultate vec moramo da dodamo async pipe da bi radilo

{{variable_async | async}}

<span style="color:tomato">| uppercase</span> 

<span style="color:tomato">| lowercase</span> 

<span style="color:tomato">| date</span>  : short :medium :long :full ...(vidi u docs)

<span style="color:tomato">| percent</span> 

.

.

.

## Services

Mozemo da ga koristimo na dva nacina:

1. Local -> Znaci da ce samo ta komponenta da ga koristi
2. Global -> Znaci da ce ga svi koristii i deliti

Service se prepoznaje ako ima dekorator(anotaciju)  <span style="color:tomato">@Injectable</span>

```java
@Injectable({
  providedIn: 'root'
})
export class TestService {

  constructor() { }
}

```

Ovako je globalno registrovana

Lokalno:

```java
@Compoenent({
	providers:[serviceName]
})
```

## Forms

Postoje 2 vrste 

### Template-Driven Form

Koristimo html,basic Form

Forme su nacin da unosimo podatke u sistem

```html
<form (submit)="onSubmit(myForm)"... #myForm>
...
</form>
```

```ts
onSubmit(form:HTMLFormElement){
    console.log(form) -> vraca nam html bukv,jer je objekat HTMLFormElement
}
```

Hocemo da dobijemo json forme

```html
<form (submit)="onSubmit(myForm)"... #myForm="ngForm">
...
</form>
```



```ts
onSubmit(form:NgForm){
    console.log(form) -> vraca nam JSON
}
```

 <span style="color:tomato">ngForm</span>-> predefinisana klasa koja sadrzi podatke o formi

ali sada nemamo input lemente u inputu, da bi to radimo moramo da 

1. Dodamo name na savki input
2. Moramo da dodamo  samo rec <span style="color:tomato">ngModel</span> -> on kaze Angularu da je to deo forme 

Pristupamo im preko ngFormInstance.value-> vraca niz vrednosti inputa

#### Validation

Na Template driven form radimo validaciju da bi mogli da je submit.

mozemo da koristimo  <span style="color:tomato">required</span> na svakom <input...> elementu da ne bi bili prazni 

<span style="color:tomato">email</span> stavimo u <input ...> i validirace emial  

svako polje koje obelezimo sa `ngModel` on ce dodavati  klase koje su kao lifecycle za formu

- ng-untouched
- ng-touched
- ng-pristine
- ng-dirty
- ng-valid
- ng-invalid

To su css klase koje se dodaju na input elemente kao deo form lifecyclea

#### Form Group

Moemo da grupisemo neke input stvari

1. Moramo da kreiramo container koji ce da ga obavija input elemente <div> neki
2. div> oznacimo sa <span style="color:tomato">ngModelGroup</span>

ako hocemo da u kodu setujemo vrednost ne mozemo samo preko forme da set(ngformnasa.value.firstName) jer ce tako samo da set u formi varijabli a ne na UI delu.

this.form.<span style="color:tomato">setValue(obj inputa)</span>

objInputa -> {

​	country:' ',

​	gender:' ',

​	group:{

​			test:' '

 		}

}

...

}

Struktura ovog objekta mora biti ista kao struktura na formi,znaci ako hocemo 1 da setujemo moramo sve ,jer struktura mora biti ista

Samo stavimo sta cemo sve da setujemo

this.form.form.<span style="color:tomato">patchValue({})</span>

Ovde stavljamo samo sta hocemo da update

### Reactive Form 

Kreiramo ih u ts klasi a ne u html-u

​	Da bi ovo koristili moramo da `importujemo ` u @NgModule **ReactiveFormsModule**

​	Idalje nam treba forma u htmlu,to mormao da napravimo.Ovde mi pravimo u kodu forme i povezujemo ih sa html-om,dok smo kod template modela direktno u html povezivali

​	sada ga napravimo u kodu <span style="color:tomato">FormGroup</span> ,i moramo da je deklarisemo sta ce sve da ima i tu namestamo opcije,te opcije du elementi FormGroup i to je

 <span style="color:tomato">FormControl(</span>

<span style="color:tomato">initialValue</span>,

<span style="color:tomato">FormControlOpitons</span>,

<span style="color:tomato">Validators</span>

<span style="color:tomato">)</span> 



Ovo ne mora da ide redosledom 

```
export class AppComponent implements OnInit{
	reactiveForm:FormGroup
	
	ngOnInit(){
		this.reactiveForm=new FormGroup({
			firstname: new FormControl(null),
			lastname: new FormControl(null),
			email: new FormControl(null),
			gender: new FormControl(null),
		
		});
	}
}
```

Bolja praksa je da pojedinacno napravimo komponente i onda ih ubacimo u FormGroup.Razlog je sto FormGroup prima AbstractFormGroup i kada mi uzmemo neku FormControll iz FormGroup nece biti FormControl vec taj apstract sto nije dobor.

```ts
  name = new FormControl(...);
  email = new FormControl(...);
  age = new FormControl(...)

  registerForm = new FormGroup({
    name:this.name,
    email:this.email,
    age:this.age,
  });
```

Kada deklarisemo ovu formu u kodu moramo da je povezemo sa formom na u html-u

<span style="color:tomato">formGroup</span> ,<span style="color:tomato">formGroupName</span> 

```html
<form [formGroup]="reactiveForm"> 
<form formGroupName="reactiveForm"> 
```

Zavisi sta bind mozemo da koristimo ime(ali onda nemamo []) ili preko celog objekata(onda koristimo [],**PREPORUCLJIVO**)

Kako ovde formu deklarisemo u kodu mi moramo to polje inputa u kodu(FormControll) da povezemo sa ovim inputom na html  i to radimo pomocu <span style="color:tomato">formControlName</span> ,<span style="color:tomato">formControl</span> 

```html
<input formControlName="ime">
<input [formControl]="ime">
```

Mi ako logujemo FormGroup imacemo pristup svim varijablama u njemo,bitne stvari za taj objekat:

- controls -> ovo je niz objekata AbstractFormControl
- errors -> niz errora
- value 
- invalid
- valid



#### Validators

Ovde imamo neke predefinisane validatore i ubacujemo ih u *FormControll*.Pristupamo pomocu klase <span style="color:tomato">Validators.neki</span>

<span style="color:tomato">min(number)</span> 

<span style="color:tomato">max(number)</span> 

<span style="color:tomato">required</span> 

<span style="color:tomato">email</span> 

<span style="color:tomato">minLength(number)</span> 

<span style="color:tomato">maxLength(number)</span> 

<span style="color:tomato">pattern(pattern)</span> 

<span style="color:tomato">nullValidator</span> 

<span style="color:tomato">compose</span>(Prima vise validatora i ujedinjuje ih) 

<span style="color:tomato">composeAsync</span> 

Svaki od validatora ima `Error Code` 

#### Custom  Async Validator

Napravimo metodu  koja vraca Observable<nesto>

```ts
emailNotValid(control:FormControl) :Observable<any>{
	const response=new Observable(()=>{
			
	});
}
```



I prosledimo ga kao argument dalje u FormControl

Ako hocemo da grupisemo neke input elemente kao pre to radimo pomocu

```ts
new FormGroup({
	personalDetails:new FormGroup({
	...
	}),
	email:new FormControl(...)
})
```

Fora je samo da nestujemo FormGroup

#### Form Array

Ovo je nacin da manupulisemo kolekcijom FormControl-a,

Mi mozemo da grupisemo FOrmControl na dva nacina:

1. FormGroup
2. FormArray

Razlika je kako su implementirani,u FormGroup su svi key-value pair,dok u array je samo deo niza

npr polje *skills*

```ts
new FormGroup({
...
skills:new FormArray([
		new FormControl(),
    	new FormControl()
    	....
	])
})
```

i da bi povezali ovo sa inputom tj ovo povezujemo sa `div-om` jer je on container koristimo <span style="color:tomato">formArrayName</span> 

```html
<div formArrayName="skills">
	<ng-container *ngFor="let skill of reactiveForm.get('skills')['controls'];index as i">
		<input formControlName="{{i}}" -> za svaki input nam treba poseban id kao,jer moramo da povezemo i ove sa formArrayom
	....
```

 <span style="color:tomato"> ValueChanges</span> je event koji se rase kada se promeni vrednot input polja

moramo da slusamo ovaj event,u zavisnosti na koje polje.

```ts
this.reactiveForm.get("firstname").valueChanges.subscribe(
	(data)=>{...}
)
```



## Parent-Child Component

Mi smo slali podatke iz .ts u template i obrnuto.Takodje mozemo da saljemo informacije iz parenta u child i obrnuto.

Za to koristimo @Input i @Output dekoratore

Imamo komponentu za listu kurseva i tu se nalazi lista tih kurseva,

Parent komponente moze da salje child-u podatke,

### @Input

Ovo stsavljamo u onu komponentu koja prima informacije,znaci u kompoennti koja prima informacije mora da ima varijabla gde ce da primi te info,e ta varijabla je oznacena sa @Input

Parent -> Child

```
@Input() all: number =10; -> znaci da ce ova varijabla koja se nalazi u Child-u da prima spolja iz parenta neku vrednost
```

kako da primi tu vrednost?

tako sto u parenent html-u ,jer ui njemu imamo html tag za child,samo u taj tag stsavimo

```
<child [imevarijable]="pavarijabla iz parenta">
```

### @Output

Ako hocemo da child salje parentu ovo koristimo.Fora je samo sto kada hocemo da saljemo varijablu iz child-a u parent mi moramo da koristimo evente

Imacemo varijablu u child-u 

```ts
extends class myChild{
	myvariable:string

    @Output
    filteredButtonChanged :EventEmitter<string>=new EventEmitter<string>();
    onRadioButtonSelectedCHanged(){
        this.filteredButtonChanged.emit(myvariable);
    }


}
```

i sada uradimo bind za evente na parentu da slusa ovu promenu

```ts
export class MyParent{
	myVariable:string='';
	
	onFilterButtonChanged( data:string){
		this.myVariable=data;
	}
	
}
```

i sada u partentu u html moramo da bind za taj event

```html
<child (filteredButtonChanged)="onFilterButtonChanged($event)" ...></child>
```

jer se salju podaci iz childa u parent kaoevent ,angular ima build in $event to ej sta salje child

## Custom events

Samo napravimo varijablu koja je tipa <span style="color:tomato">EventEmmiter<T></span>

```ts
extends class myChild{
	myvariable:string

    @Output
    filteredButtonChanged :EventEmitter<string>=new EventEmitter<string>();
    onRadioButtonSelectedCHanged(){
        this.filteredButtonChanged.emit(myvariable);
    }


}
```

Emit se preko funkcije emit(value)

## Template Reference Variable

Ovo je referenca DOM elementa,da bi neku varijablu napravili koristimo <span style="color:tomato">#IMEVARIJABLE</span>

Kada neki DOM element ima ovo mi mozemo u html kodu da ga referenciramo

```
<input type="text" #myVarialbe> -> sada mozemo myVariable da koristimo svuda u DOM-u ,i to je ceo taj 									DOM element
<p>{{myVariable.value}}</p>

<button (click) ="sayHello(myVariable)">Say Hello</button> -> kada se desi klik pozivamo ovu metodu i prosledjuemo referencu na input
.ts
sayHello(inputEle:HTMLInputElement){

}
```

## Dekoratori

#### @ViewChild

ovo su anoracije,gore smo koristili metodu da bi prosledili DOM element,ali sta ako hocemo da pristupimo DOM elementu pre nego sto se desi taj event?Koristimo <span style="color:tomato">@ViewChild("templateReferenceVariable")</span> 

Stavimo ovo na neku varijablu u ts fajlu i on ce da referencira DOM element,tip je <span style="color:tomato">ElementRef</span>

```
<input #myInput>
```

.ts

```ts
exdends class AppComponent{
	@ViewChild("myInput") myDOMElement:ElementRef;
}
```

Fora je samo da u @ViewChild(ovde tavimo template reference variable)

da bi pristupili tom elementu u kodu idemo .nativeElement

#### @ContentChild

Hocemo da pristupimo ovom <p> u child komponenti koja je ngcontent

![image-20230630105133384](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230630105133384.png)

Ako hocemo da pristupimo komponenti koja ima ono inner tj one ngContent stvari,tu ne mozemo da radimo viewChild.

```html
<app-demo>
	<p #myVarialbe>cao</p>
</app-demo>
```

e sada ne mozem oda pristupimo ovome p preko ViewChild jer je unutrasnji tag za app-demo.

```
@ContentChild('myVariable') ime:ElementRef
```

kada bi koristili ovu varijablu necemo imati code compeation jer  @ContentChild vraca  any ,pa necemo znati o cemu je rec,ovo mozemo da prevazidjenom uvodjenjen `QueryList`

```ts
  @ContentChildren(TabComponent) tab:QueryList<TabComponent> =new QueryList<TabComponent>();
```



#### @ContentChildren(Tipklase)

Ovo ej ako imamo neku komponentu koja prima preko ng-content-a vise dece ,da bi dobili pristup svoj deci koristimo ovu direktivu

```ts
 @ContentChildren(TabComponent) tabs={};
```

Samo moraimo da kazemo o kojoj klasi je rec

#### @HostListener

Slusa event na host element,koristimo ga u kombinaciji sa `directive`

Definisemo ga na funkciji ,i kazemo za koji event.Ta funkcija ce se izvrsiti ako se taj event desi

```ts
@Directive({
    selector: '[appHover]'
})
export class Test{
	constructor(private element:ELementRef,private renderer:Rederer2){
		
	}
	
@HostListener('mouseenter') onMouseOver(){
		this.renderer.setStyle(this.element.nativeElement,'margin','30px 30px');
		this.renderer.setStyle(this.element.nativeElement,'padding','5px 5px');
         this.renderer.setStyle(this.element.nativeElement,'transition','0.5s');
    }
}
```

Mi smo ovde hardkodovali vrednosti ,mi hocemo da komponenta kada stavi directive da ona ubaci te vrednosti,da se dinamcki odredjuju

```ts
@Directive({
    selector: '[appHover]'
})
export class Test{
	constructor(private element:ELementRef,private renderer:Rederer2){
		
	}
@Input() defaultColor:string='transparent';
	
	
@HostListener('mouseenter') onMouseOver(){
		this.renderer.setStyle(this.element.nativeElement,'backgroundColor',defaultColor);
    }
}
```

Moramo da stavimo input da bi primio vrednost

```html
<p appHover [defaultColor]="'#GEW214'">
</p>
```



#### @HostBinding

na host element,koristimo ga u kombinaciji sa `directive`

Mi kazemo za sta ce da se bind vrednost varijable,ovde se ona bind za style.backgoundColor.Sto ide style,pa jer se radi o ElementRef-u,mi ovo samo stavimo uHTML element

```ts
@Directive({
    selector: '[appBetterHighlight]'
})
export class Test{
	constructor(private element:ELementRef,private renderer:Rederer2){
		
	}
	
@HostBinding('style.backgroundColor') backgounrd:string='transparent';
}
```

```
<p [appBetterHighlight]>cao</p>
```



## Renderer2

Ovo koristimo da manipulisemo DOM elementima u kodu,ne u samom html-u

MI Rendere2 mozemo da ubacimo u konstruktor

```ts
export class Test{
	constructor(element:ELementRef,renderer:Rederer2){
		this.renderer.nesto
	}
}
```

## Observables

Ovo je jako bitan koncept ,zasniva se da angular slusa dolazece promene i kada se neka desi onda on reaguje

Ovo je asinhrono i nece da blokira,slusace na neke promene i reagovace.

Jako je bitno ovo jer se pomocu ovoga salju network pozivi.

Pre su se koristili `Promise` a sada `Observable` jer je novo

Observable salje podatke u pakete,strimuje data,ovo je deo RxJS-a

Mi kada dobijemo observable,mi moramo da se  <span style="color:tomato">subscribe(next,error,complete)</span>,observable ce periodicno da vraca nesto,i zavisi sta je to nesto bice handlovano od strane ove 3 funkcije 

```ts
myObservableInstance.subscribe(next,error,complete)

myOservable.subscribe(
(value)=>{},
(error)={},
()=>{}
)
```

Da napravimo novu observable instancu(obicno en pravimo vec nam je provide npr iz http poziva mi se samo subscribe)

```ts
myObservable=new Observable((observer)=>{
	    observer.next(1);
    	observer.next(2);
    	observer.next(3);
    	observer.next(4); -> Obicna Next function ce da handle ovaj emit
    	observer.error("...") ->Error function ce da handle ovaj emit
    observer.complete() -> complete function ce da handle ovaj emit
})
```

Ovi observable mozemo da primenimo operatore.

unsubscribe from observable

samo pozovemo funkciju unsubscribe

```
myobservable.unsubscribe()
```



#### pipe

```
myObservable.pipe
```

## Routing

Da bi radili routing moramo da definisemo varijablku tipa <span style="color:tomato">Routes</span>

```ts
const  routes:Routes=[
 ...
]
```

i tu definisemo niz objekata tipa Route <span style="color:tomato">Route</span>.On ce ove rute da **gleda redom** jedna ispod druge i zavisi gde ce da match,ako ne match ni jednu iz niza onda ne postoji

```ts
 {path:'Home',component: AppComponent}
```

Moramo da definisemo path u utl-u pomocu kog se pristupa,takodje  i komponentu koja je na tom putu da se priakze

Samo ovo nije doboljno da desfinisemo,moramo da ga ukljucimo u `Module` 

```ts
@NgModule({
...
imports:[
	RouterModule.forRoot(routes)
]
})
```

Ovde moramo ubaciti taj niz ruta

i zadnji korak je da ubacimo u root compoent(Nasa glavna kompoenenta koja sve prikazuje)

```html
<router-outlet></router-outlet>
```

Znaci koraci su:

1. Definisati objekat `Routes`
2. Ukljuciti u `NgModule`
3. Ukljuciti u `app.component` rute

Objekat tipa Route

```
{
	path:'' ->URL gde radi navigaciju
	redirectTo: "URL" -> ovo je URL gde ce da redirect
	pathMatch:'full' -> na koji nacin da match url 
	component: ComponentName -> na koju kompoenentu da posalje
	children:[{}...]
}
```

```
<a hreaf="Home">Home</a>
<a hreaf="About">About</a>
<a hreaf="Contact">Contact</a>
<a hreaf="Courses>Courses</a>

```

Ako ovako samo namestimo linkove,ANgular po defaultu refresh celu stranu,a to ne zelimo.Zelimo da samo zamenjuje komponeente ,ne da reload sve odjednom,zato umesto `href` koristimo direktivu  <span style="color:tomato">routerLink</span>

```
<a routerLink="Home">Home</a>

```



### 404 Route

Ako ne match ni jednu rutu bacice error,to moramo mi da handle jer on ide redom i match rute,zato nam 404 ruta treba biti zadnja

Napravimo novu komponentu koja ce biti `PageNotFount`

```
.
.
.
{path:"**",component:ErrorComponent} -> bukvalno ** ako se bilo sta match na kraju onda je to error,jer je sve prethodno prosao i nista nije match
```



#### Styling active link

Kada nam je link aktivan(kada smo uradili navigaciju tu) zelimo da ima odredjeni stil.

Napravimo css klasu koja ce se dodeljivati kada je link active

```css
.active{
	...
}
```

opet koristimo direktivu <span style="color:tomato">routerLinkActive</span>

```html
<li routerLinkActive="active"><a routerLink="">Home</a></li>
```

Ova direktiva **toggle** css klase kada je link aktivan.

Moramo paziti jer parent stvari ce biti toggle ako je dete toggle

npr:

{path:"",component:Home} -> URL za ovo je samo localhost:4042

{path:"Contact,:component:Contact} -> URL za ovo je localhost:4042/contact

e sada vidimo da je ovaj prvi link gore sadrzan u ovom drugom,tako da kada je contact aktivan bice kativan i ovaj home a to necemo

Ovo mozemo da sklonimo:

1. Da paizmo na imenovanje ruta

2. Da koristimo  <span style="color:tomato">routerLinkActiveOptions</span>

   ```html
   <li routerLinkActive="active" routerLinkActiveOptions="{exact:true}"><a routerLink="">Home</a></li>
   ```



#### Absolute ,Relative routes

DO sada nismo stavljali / u ruti,kada stavimo / to znaci da je apsolute path,znaci taj path se **app end** na root url



### Programmaticly Routing

DO sada smo koritsili routerLink za navigaciju,mozemo to da radimo u kodu sve

To radimo pomocu DI klase <span style="color:tomato">Router</span>

​	navigateByUrl(string)

​	navigate([string])

```ts
this.router.navigate(['Home'])
this.router.navigateByUrl('Home')
```

ove metode uvek koriste **absolute path**,moramo paziti na to

ako hocemo da uradimo **relative path** moramo da  inject klasu <span style="color:tomato">ActivatedRoute</span>.Ova klasa nam daje informacije o trenutnoj aktivoj ruti

```ts

this.router.navigateByUrl('Home',{relativeTo:this.activatedRoute});
```

ovako bi radilo relative,jer mi moramo da posaljemo ActivatedRoute

### Passing Parameters

Ovo je primer kada radimo detalje nekog proizvoda,imamo listu proizvoda i hocemo da vidimo detalje tog proizvoda

Moramo da definisemo rutu koja ce da ima dinamicki parametar,to radimo pomocu <span style="color:tomato">:ime</span>

```
{path:'Courses/Course/:id',component:CoursesComponent}
```



sada nam treba da u htmlu kada se klikne na nesto da se radi navigacija.

```ts
export class CourseCompnent implements OnInit{
	course;
	courseId;
	
	constructor(private activeRoute:ActivatedRoute){}
	
	ngOnInit():void{
		this.courseId=this.activatedRoute.snapshot.paramPam.get('id');
		//call service and pass id to get spesific course
	}
}
```



i na svakom elementu u listi moramo imate:

```html
<button routerLink="Courses/Course/{{course.id}}">Show Deatils</button>
```

fora je da samo uradili interpolation mapping 

#### Dinamicka promena parametra

Ako uraidmo navigaciju ka nekom kursu sa idjem 101,i u toj strani detalja imamo dugme koja nas vodi na kours 102,ako to kliknemo nece se nitsta desiti.Ovo je dafault ponasanje angular-a,obicno jer smo uzeli param u ngOnInit  sto se poziva samo jednom.Tako da se komponenta ne menja vec se menja samo id pa se ngOnInit ne poziva opet jer on reuse vec postojacu.

Kako se taj id menja mi ne mozemo da koristimo snapshop jer to je izvrsavanje samo jedon,vec **observable**

```ts
export class CourseCompnent implements OnInit{
	course;
	courseId;
	
	constructor(private activeRoute:ActivatedRoute){}
	
	ngOnInit():void{
		this.courseId=this.activatedRoute.paramMap.subscrive(
        	(param)=>{
                this.courseId=param;
            }	
        )
		//call service and pass id to get spesific course
	}
}
```

### Query Parameters

U Angularu su rute iste kao i URL rute

Query param je optional param koji se dodaje na kraju url-a

```
localhost:4200/products?id=12345&name=iphone
```

Da bi prosledili param samo uradimo ovo,key-value pair je 

```html
<button ... queryParams="{edit:true,value:key}"></button>
```

Moemo ovo da uradimo i u programu preko koda:

```ts
export class CourseCompnent implements OnInit{
	course;
	courseId;
	
	constructor(private router:Router){}

	navigate(){
		this.router.navigate(['/Courses/Course',this.courseId],(queryParams: 			{edit:true} ))
	}
}
```

Fora je da kada pozivamo ovu funkciju navigate samo prosledimo queryParams kao dodatan argument.To je objekat koji prima key-value pair



i sada mozemo da uzmemo parametar 

```ts
this.activeRoute.snapshop.queryParamMap.get('edit');
```

ali ovo nece da radi jer kada se init component nece vise imati taj query param,bitno je da radimo sa observable da se osiguramo

```ts
this.activeRoute.queryParamMap.subscribe((data)->{

})
```

#### Fragments

Fragment  u rulu je link u kome se brzo premestamo na sekciju u strani.Oni se pisu sa  <span style="color:tomato">#</span>

da bi odradili navigaciju na taj frament(da se uabci fragment u url) potrebno je da definisemo 

<span style="color:tomato">fragment="ime"</span>

```
localhost:4200/Products#service
```



```
<a routerLink="" fragment="id_of_div_or_component">about</a>
..
.
.
<aboutCompoennt id="about"></aboutCompoennt>
```

Ovako ga prenosimo u url,a kako da ga uzmemo u kodu?

Mi kada kliknemo na taj deo koji sadrzi fragment u url ce se dodati #ImeFragmenta pa to raidmo preko activatedRoute

Pomocu activated route fragment deo vraca observable

```ts
export class AppComponent imlpements OnInit{

	constructor(private activeRoute:ActivatedRoute){}
    ngOnInit(){
		this.activatedRoute.fragment.subscribe((data)=>{
		
		})
    }
}
```

### Child Route

Ovo se jos zovu i nested routes.

npr imamo 

{path:'Courses',component:CourseComponent}

{path:'Courses/Course/:id',component:CourseDetails}

Vidimo da imaju istu bazu u putanju pa mozemo da uradimo sa nested routes

```ts
{path:'Course',children:[
	{path:'Course/:id',component:CourseComponent}
]}
```

fora je da ubacimo childern

### Routes In Module

Kako bi imali lepu strukturu onda pravimo novi module za rute

napravimo `app-routing.module.ts`

```
	const appRoute:Routes=[
	...
	]
@NgModule({
	imports:[
		RouterModule.forRoot(appRoutes);
	],
	exports:[
		RouterModule
	]
})
export class AppRoutingModule{

}
```



i onda mozemo da izbrisemo u @NgModule u imports ono .forRoot(...) jer smo ga premestili ovde sada da bude.

Onda samo dodamo u imports onvaj novi module

### Route Guard

Pomaze nam da osiguramo rutu,ili da izvrsimo neku akciju pre same navigacije

npr ne mozemo da pristupimo ruti dok se ne login

Mozemo koristiti:

- Da potvrdimo navigaciju
- Da pitamo da li zeli da scuva podatke
- Da dozvoli pristup odredjenim delovima aplikacije
- Da validira parametre pre navigacije
- Da fetch neke podatke pre nego sto display view

Postoje neki predefinisani tipovi:

1. <span style="color:tomato">CanActivate</span>  -> Da li moes da pristupis ruti

2. <span style="color:tomato">CanDeactivate</span> -> Da li moze da izadje iz trenutne rute i ode drugde negde(navigate away,back dugme kao )

3. <span style="color:tomato">Resolve</span>-> Slizi nam da ucitamo neke podatke pre nego sto se izvrsi navigacija,ovo vraca data koja se ucitava

   ```
   {path:'...',component:'',resolve:{courses:CourseResolveService}}
   ```

   Ovo je malo drugacije nije samo da moramo da kazemo koji Route Gouard ima vec te podatke koje vraca resolve metoda cuvamo u varijablu courses,i onda uzmemo te podatke 

   ```ts
   this.ActivatedRoute_Intance.snapshot.data['courses'] 
   ```

   

4. <span style="color:tomato">CanLoad</span>

5. <span style="color:tomato">CanActivateChild</span> ->Da li mozes da pristupis deci (rutama) stavlja se na parent route



1) Moramo da napravimo RouteGuard service koji implementira neki od ovih 5 tipova

2) Moramo da taj service ubacimo u module u providers

3) onda moramo taj service da primenimo na ruti

   ```ts
   {path:'...',component:'...',canActivate:[MY_SERCICE]}
   ```



### Navigation Events

- NavitaionStartEvent
- RoutesRecognisedEvent
- GouardCheckStartEvent
- GaurdCheckEndEvent
- ActivationStarteEvent
- ActivationEndEvent
- NavigationEndEvent

Mi se mozemo sub na neke od ovih eventa i da run neki deo koda

Npr mozemo da display loading image dok se ucitavaju podaci ka stranici,

```ts
route : Router
this.router.events.subscribe((navigationEvent:Event)=>{
	if(navigationEvent instanceof NavigationStart){
        ....display loading
    }
})
```

znaci da bi loading display bio na kompoennti sa koje idemo na neku drugu.

## Fore:

Ako funkcija vraca false onda ce da prevent defaults ono ponasanje

ng add redi slicno sto i npm install samo sto nam daje bolje instrukcije
