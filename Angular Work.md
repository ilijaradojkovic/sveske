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