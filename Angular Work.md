# Angular Work

## Search

![image-20230630141533203](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230630141533203.png)

Imamo ovakvu situaciju gde `Search` komponenta je posebna i ona sadrzi "SearchText" varijablu u kojoj je smesten text.

Ovde moramo da imamo @Output varijablu koja ce da emituje vrednost search-a,sada kako ce se taj event pozivati zavisi

1) Pozivacemo ga kada user klikne dugme `Search`

   ```ts
   @Component({
     selector: 'app-search-bar',
     templateUrl: './search-bar.component.html',
     styleUrls: ['./search-bar.component.css']
   })
   export class SearchBarComponent {
   
     _searchText:string="";
   
   
   
     @Output()
     filteredButtonChanged :EventEmitter<string>=new EventEmitter<string>();
     onSearchClicked(text:String){
       console.log(this._searchText);
   
       this.filteredButtonChanged.emit(this._searchText);
     }
   }
   ```

   ```html
   <button (click)="onSearchClicked(search.value)" class="btn btn-lg btn-success" type="submit">Search</button>
   
   ```

   Pa kada se klikne na dugme on ce da pozove funkciju koga ce da emituje taj event.Taj event mora da se slusa od strane pernt-a

```html
      <app-search-bar (filteredButtonChanged)="onFilterButtonChanged($event)"></app-search-bar>

```

2. Mozemo da registrujemo onChangeText

   

```html
            <input (ngModelChange)="onSearchClicked($event)" #search name="_searchText" [(ngModel)]="_searchText"  class=" form-control form-control-lg form-control-borderless" type="search" placeholder="Search topics or keywords">

```

na input polju i kada promeni text pozivace se ova funkcija koja ce da podize event

## Reactive Form

Hocemo da napravimo formu za register

```html
<form [formGroup]="registerForm" (ngSubmit)="register()">
  <!-- Name -->
  <div class="mb-3">
    <label class="inline-block mb-2">Name</label>
    <app-input [formControl]="name" placeholder="Enter Name" type="text">

    </app-input>
  </div>
  <!-- Email -->
  <div class="mb-3">
    <label class="inline-block mb-2">Email</label>
    <!--kad hocu da setujem objekat mora [] a kada je string ili number onda ide bez toga []-->
    <app-input [formControl]="email" placeholder="Enter Email" type="email">

    </app-input>
  </div>
  <!-- Age -->
  <div class="mb-3">
    <label class="inline-block mb-2">Age</label>
    <app-input [formControl]="age" placeholder="Enter Age" type="number">

    </app-input>
  </div>
  <!-- Password -->
  <div class="mb-3">
    <label class="inline-block mb-2">Password</label>
    <app-input [formControl]="password" placeholder="Enter Password" type="password">

    </app-input>
  </div>
  <!-- Confirm Password -->
  <div class="mb-3">
    <label class="inline-block mb-2">Confirm Password</label>
    <app-input [formControl]="configrm_password" placeholder="Confirm Password" type="password">

    </app-input>
  </div>
  <!-- Phone Number -->
  <div class="mb-3">
    <label class="inline-block mb-2">Phone Number</label>
    <!--Moramo da instaliramo input max preko "npm i ngx-mask"-->
    <app-input [formControl]="phoneNumber" placeholder="Enter Phone Number" format="(000)000-0000">

    </app-input>
  </div>
  <button
    [disabled]="registerForm.invalid"
    type="submit"
    class="

          block w-full bg-indigo-400 text-white py-1.5 px-3 rounded transition
                  hover:bg-indigo-500
          disabled: opacity-50"
  >
    Submit
  </button>
</form>

```

Za svaku input komponentu pravim novu component i tu dinamcki je koristimo vise puta

```html
<input
  [formControl]="formControl"
  [type]="type"

  class="block w-full py-1.5 px-3 text-gray-200 border border-gray-400 transition
                    duration-500 focus:outline-none rounded bg-transparent focus:border-indigo-400"
  [placeholder]="placeholder"  />
<ng-container *ngIf="formControl.dirty && formControl?.invalid">
  <!--ng-content,ng-template(content must be conditionally rendered) ovo je invisible container da ne menjamo kontent sa <div> elemntom -->
  <p *ngIf=" formControl?.errors?.['required']" class="text-red-400">
    Field is required.
  </p>
  <p *ngIf="formControl?.errors?.['minlength']" class="text-red-400">
    The value must be more then 3 characters.
  </p>
  <p *ngIf="formControl?.errors?.['email']" class="text-red-400">
   Email is not valid.
  </p>
  <p *ngIf="formControl?.errors?.['min']" class="text-red-400">
   Number cant be less then 18
  </p>
  <p *ngIf="formControl?.errors?.['max']" class="text-red-400">
   Number cant be more then 120
  </p>
  <p *ngIf="formControl?.errors?.['pattern']" class="text-red-400">
   Pattern does not match(Must contain 1 letter and 1 number).
  </p>

</ng-container>
```

Dinamicki unostimo kog ce `tipa` biti(text,number,email...)

DInamicki unostimo samu `FormControl` da bi se bind

Dinamickiu nosimo `placeholder`

Takodje ovde pisemo i errore za field,samo sto necemo sve errore da prikazujemo,vec pomocu te form controll proverimo da li ta vrsta vlidatora postoji,i ako je netacna,pokrijemo sve slucajeve.