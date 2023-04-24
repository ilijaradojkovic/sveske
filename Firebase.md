# Firebase

Da bi radili  bilo sta sa Firebase-om moramo napraviti projekat i onda njega koristimo.

![image-20230424150726419](C:\Users\Ilija\AppData\Roaming\Typora\typora-user-images\image-20230424150726419.png)



Fora je da odemo ovde na `Admin SDK` i tu ce nam dati primer koda koji mi moramo da izvrsimo da bi radio Firebase 

## Messaging

Ovo je funkcionalnost koji nam Firebase nudi za slanje poruka (`Notifications`)

```java

gcp:
  firebase:
    service-account: classpath:firebase.json
        
        
        
        

@Value("${gcp.firebase.service-account}")
    public Resource firebaseConfig;
    @Bean
    GoogleCredentials googleCredentials() {
        try {
            if (firebaseConfig != null) {
                try( InputStream is = firebaseConfig.getInputStream()) {
                    return GoogleCredentials.fromStream(is);
                }
            }
            else {
                // Use standard credentials chain. Useful when running inside GKE
                return GoogleCredentials.getApplicationDefault();
            }
        }
        catch (IOException ioe) {
            throw new RuntimeException(ioe);
        }
    }

    @Bean
    FirebaseApp firebaseApp(GoogleCredentials credentials) {
        FirebaseOptions options = FirebaseOptions.builder()
                .setCredentials(credentials)
                .build();

        return FirebaseApp.initializeApp(options);
    }

    @Bean
    FirebaseMessaging firebaseMessaging(FirebaseApp firebaseApp) {
        return FirebaseMessaging.getInstance(firebaseApp);
    }
```



Bitno je da za messaging koristimo instancu <span style="color:red">FirebaseMessaging</span> i sa njom cemo sve da radimo,zato smo definisali bean kako bi je samo autowire

Pomocu ovog Resource samo uzmemo `firebase.json` fajl za auth na firebase .

 Za slanje poruke koristimo <span style="color:red">Message</span> klasu

Message klasa je osnova za slanje poruke,nju pravimo preko builder-a

```java
     Message message= Message.builder()
                .setNotification() -> Ovo je objekat,sama notifikacija 
                .setTopic() -> na koji topic saljemo
                .setToken() -> token uredjaja
                .setCondition() -> izvrsi se samo ako je ovo true
                .putData()
                .putAllData()
                .setAndroidConfig()
                .setWebpushConfig()
                .setFcmOptions()
                .build();

 Notification.builder()
                .setBody() -> sadrzaj poruke
                .setImage() -> slika poruke
                .setTitle() -> naslov poruke
          		.build()
```

Mi slanje poruke radimo preko `topic` i preko ` token` 

token -> ovo svaki uredjaj dobije kada se registruje na firebase kako bi iskoristio listen metodu.Znaci nasa android app npr ce da se registruje dobice token i nama ce da prosledi i mi cemo znati gde da saljemo

topic -> klasican topic

Takodje ako imamo vise tokena mozemo da posaljemo odjednom,ovde u obican message to nema,moramo da koristimo 

<span style="color:red">MulticastMessage</span>,ovde nemamo topic

```
  MulticastMessage.builder()
                .addAllTokens() -> dodaje listu tokena,tako da salje svima u listi
                .setNotification()
                .setAndroidConfig()
                .setWebpushConfig()
                .build()
```

