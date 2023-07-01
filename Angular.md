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
  selector 
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

<span style="color:tomato">declarations</span> -> stavljamo komponente da ih register

<span style="color:tomato">imports</span>-> register other modules

<span style="color:tomato">providers</span>->

<span style="color:tomato">bootstrap</span> ->

<span style="color:tomato">})</span>

## Directives

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
	transform(value:string,params):string{
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

Forme su nacin da unosimo podatke u sistem

```html
<form ...>
...
</form>
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

