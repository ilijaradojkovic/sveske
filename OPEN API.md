# OPEN API

Ovo je 3.0 verzija za dokumentovanje naseg api-ja.

Dodamo dependency 

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.14</version>
</dependency>
```

i moramo da napisemo @OpenAPIDefinition kako bi enable da radi lepo.Kako imamo u app @RestCOntroller on ce automatski da ga prepozna i izgenerisace html kako bi to lepo izgledalo

pokrenemo app i idemo na localhost:9000/swagger-ui/index.html 

port je drugaciji samo jer zavisi kako pokrenemo na koji port app

Mi cemo imati izgenerisan fajl pomocu ovoga,ali mi mozemo ovo jos da nadogradjujemo sa anotacijama

- <span style="color:#ff4d4d">@Operation</span>
- <span style="color:#ff4d4d">@ApiResponses</span>
- <span style="color:#ff4d4d">@ApiResponse</span>
- <span style="color:#ff4d4d">@Content</span>



```java
@Operation(method,description,summary,)
```

```java
@ApiResponses(
        @ApiResponse(description,responseCode,headers,content)
)
```

```java
@Content(mediaType,exampleObject)
```