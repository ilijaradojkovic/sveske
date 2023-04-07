U @Column ne moze enum iako postoji enum sa parametrima ,jer ovo se ucitava pre klase enum ,jedino static final varijable mozemo da ubacimo.

Ako je pom onako siv znaci da ga je intelij automatski disable moramo da idemo na modul desni klik u maven view i da ga enable.

Pisalo mi na kafki class does not exists to je jer kafka kesuje stvari,pa sam izbrisao ceo container i pokrenuo opet

org.mockito.exceptions.misusing.UnnecessaryStubbingException:  mi se pojavio kada sam mock neku metodu koja se nije ni pozivala,jer baca exception pre toga pa nema potrebe da mock
Kod mapera moze metoda da baca excetion kada pozovemo expression neki

Imao sam problem prilikom pozivanja endpointa,on vraca [ {...}] pa sada ja to moram u klasu da pretvorim,kako je ovo niz klasa samo pretvorimo sa klasata[].class u mom slucaju je bilo CityResponse[].class jer je vracao niz tih objekata.



Kod monga sam pisao u document kao collation umesto collection pa javlja Field locale gresku

Ako imamo value on ne sme biti de oconstructora ,trazice da mi provide instancu,mora da bude van konstruktora sa @RequiredArgs

Imao sam problem u liquibase kada sam zeleo jhipster app da update bazu.Pisalo mi je da ne moze da dobije change lock .Pa sam nasao u mojoj bazi tabelu databasechangelocklog i tu sam samo stavio na lock=false i radilo je 

Imao sam opencsv i liquibase.Liquibase ima isto opencsv dependency pa sam pomesao dependencies pri importu

InputStream se cita samo 1 moramo da ga mark(Integer.Max) ovo ga oznaci na pocetak pa da ga resert() i tako ga vratimo na mark.Mulitipart file sam imao gde sam 2 puta citao inputstrema pa sam morao da ga reset 