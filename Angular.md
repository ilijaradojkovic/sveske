# Angular

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

Operatori u HTML-u.

Mi imamo mogucnost da pisemo neke operatore,funkcije u html-u kao kada smo raidli sa thymeleaf-om

<span style="color:tomato">*ngIme</span> ovo je opsta sintaksa

### if

<span style="color:tomato">*ngif=ifLogic</span>

​	Prikazuje ili sakriva komponente u zavisnosti da li je true ili false

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



## Angular Lifecycle

Angular prolazi kroz razne faze tokom rada,mi te interfejse moramo u compoennnt da bi overide taj lifecycle i da ga koristimo.

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

Ako hocemo da child salje parentu ovo koristimo.

Imacemo varijablu u child-u 

```
extends class myChild{

myvariable:string

}
```

