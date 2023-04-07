# Liquibase

Ovo je tool koji ugradimo u app,kao plugin ubacimo 

```xml
<plugin>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-maven-plugin</artifactId>
    <version>4.16.0</version>
    <configuration>
        <propertyFile>liquibase.properties</propertyFile>
    </configuration>
    <dependencies>

        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.2</version>
        </dependency>

    </dependencies>
</plugin>
```

Ovim cemo ukljuciti liquibase kao plugin i moci cemo da koristimo ovaj tool.

Ovaj tool je namenjen da verzionisemo bazu,da joj jakse manipulisemo.

Prvo moramo uraditi konfiguraciju fajla da bi podesili bazu koji ce liquibase da gadja,i gde ce da pise odredjene promene.

Napravimo strukturu u resouce fajlu:

```ini
db

	changelog

					data

					diffs

					patches
```



<span style="color:red">changelog</span> -> ovo ce sadrzati glavne konfiguracione fajlove

sve fajlove cemo pisati u yml formatu

Napravimo dva fajla u changelog-u:

- db.changelog-diff.yml
- db.changelog-master.yml

Ova dva fajla su nam kljucna

Master fajl je glavna konfiguracija

changelog-diff nam je fajl koji kada izvrsimo komandu diff upisuje razlike izmedju trenutnog stanja u bazi i nasih fajlova u app.

```yaml
databaseChangeLog:
  - includeAll:
      path: db/changelog/diffs
  - includeAll:
      path: db/changelog/patches
```

## Patches file

Ovo je konfiguracioni fajl za data

```yaml
databaseChangeLog:
  - changeSet:
      id: load_countries_from_csv_into_database
      author: Ilija
      changes:
        - loadData:
            encoding: UTF-8
            file: ../data/countries.csv
            relativeToChangelogFile: true
            tableName: country
            usePreparedStatements: true
```

Ovo je konfiguracija da ucita sve podatke iz data fajla i napuni bazu dummy vrednostima,tj vrednostima iz csv fajla u data folderu.

## Data file

 Ovo se nalazi unutar changelog fajla, i sluzi nam da ucitamo neke podatke u bazu,time punimo bazu

Tu ubacimo csv vrednosti koje ce liquibase sam da procita.

## Diff file

Ovo se nalazi unutar changelog fajla, i sadrzi sve promene koje smo nacinili u bazi

## Commands

Pre svake komande moramo uraditi <span style="color:red">maven clean install</span>

 <span style="color:red">diff</span> -> liquibase diff je komanda koja radi razliku izmedju stanja baze u aplikaciji (to su svi fajlovi iz diff foldera) i stanja u bazi

 <span style="color:red">update</span> ->liquibase update je komanda koja uzima sve fajlove iz diff foldera i izvrsava ih nad nasom bazom

