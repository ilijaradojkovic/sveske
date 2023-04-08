 

# LOMBOK

 

Lombok je biblioteka koju koristimo kako ne bi pisali dosta duplog koda.Taj dupli kod se odnosi na pisanje koda kod kalsa,kada kreiramo klase moramo za svako polje da generisemo get i set metode kao i tostring i equas,ovo sve radi umesto nas.

Lomok je biblioteka koja nam daje anotacije koje stavljamo na klasi i time olaksavamo rad

 

<span style="color:#ff6347">@NonNull</span>

  *Variable*

 

Ovo zamenjuje if(variable==null) proveru,baca null pointer exception ako je null.Ovo bukvalno menja sve one null provere samo anotiramo tu varijablu.

 

<span style="color:#ff6347">@Cleanup</span>

*Variable*

Oznacavamo varijable koje implementiraju interface closeable,tj one klase koje se moraju zatvoriti.Ne moramo rucno da zatvaramo 

@Cleanup InputStream in=…

<span style="color:#ff6347">@Getter</span>

*Variable /Class*

Generise getter za varijablu,ako stavimo na class sve varijable ce imati automatski generisan getter

 

<span style="color:#ff6347">@Setter</span>

*Variable/Class*

Generise setter za varijablu,ako stavimo na class sve varijable ce imati automatski generisan setter

 

<span style="color:#ff6347">@EqualsAndHashCode</span>

*Class*

 

<span style="color:#ff6347">@Data</span>

Variable/Class

Ovo je kao da smo stavili @Getter @Setter u isto vreme,dodaje jos ToString.

 

<span style="color:#ff6347">@ToString</span> **(**

**includeFieldNames** :Boolean -> po default je ovo true,i onda generise string kao FieldName=value,a ako je false bice samo value i necemo znati imena za ta polja

**doNoUseGetters** :Boolean -> po default je ovo true,nece koristiti get metode polja vec samo njihove vrednosti direktrno

callSuper :Boolean ->da uz svoj string pozove super.toString,default je skip

**)**

*Class*

 

<span style="color:#ff6347">@NoArgsConstructor</span>**(**

**Force :**Boolean-> Ako imamo polja ovo im dodaje default value

**AccessLevel** :AccessLevel

**)**

 

<span style="color:#ff6347">@RequiredArgsConstructor</span>**(**

**StaticName** :String ->staticko pozivanje da se klasa napravi (ako je of npr class.of(elementsinconstructor)

**AccessLevel** :AccessLevel

**)**

*class*

Primenjuje se samo na polja koja su final ili @NonNull,tj napravice konstruktor koji sadrzi samo ta polja,

 

<span style="color:#ff6347">@AllArgsConstructor</span> **(**

**StaticName** :String ->staticko pozivanje da se klasa napravi (ako je of npr class.of(elementsinconstructor)

**AccessLevel :AccessLevel**

**)**

**
 @Value**

*Variable/Class*

Ovo je isto kao @Data samo sto je immutable,varijable ce biti final

 

 

 

 

<span style="color:#ff6347">@Builder</span>**(**

**BuilderMethodName** :String->Ovo je metoda koju pozivamo da dobijamo bilder,po default je Builder

**BuildMethodName** :String-> Ovo je metoda koja ce da napravi obj ,po default je build

**AccessLevel :AccessLevel**

**)**

 

<span style="color:#ff6347">@Singular </span>

Oznacimo kolekciju u Builder klasi,dobicemo metode za add i remove iz te kolekcije ,da ih ne pravimo sami.Jer po nekoj praksi ne bi trebalo da imamo direktan pristup kolekciji vec preko metoda da dodajemo i brisemo iz nje.

 