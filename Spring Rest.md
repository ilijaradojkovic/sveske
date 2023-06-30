# Spring Rest

## HTTP

HTTP(HyperText Transfer Protocol) je protokol aplikativnog sloja koji koristi TPC transportni sloj kako bi odrzao konekciju.Sluzi za prenos poruka preko interneta u cistom tekstu.



**Session state** is also known as **Stateless state**. HTTP is a stateless protocol. In the session state, the client and server just know about each other only during the current request. If the connection is closed, and two computers want to connect again, they need to provide information to each other as a new connection, and the connection is handled as the very first one.



An HTTP response contains the following things:

1. Status Line
2. Response Header Fields or a series of HTTP headers
3. Message Body

HTTP 1.1 ->prva popularna verzija 

### HTTP 2 

Over the years, web pages became more complex. Some of them were even applications in their own right. More visual media was displayed and the volume and size of scripts adding interactivity also increased. Much more data was transmitted over significantly more HTTP requests and this created more complexity and overhead for HTTP/1.1 connections. 

In HTTP/1.0, the connection is closed after a single request or response pair. In HTTP/1.1, a **mechanism** was introduced, which is known as keep-alive-mechanism. In this mechanism, a connection could be reused for more than one request

The HTTP/2 protocol differs from HTTP/1.1 in a few ways:

- It's a binary protocol rather than a text protocol. It can't be read and created manually. Despite this hurdle, it allows for the implementation of improved optimization techniques.
- It's a multiplexed protocol. Parallel requests can be made over the same connection, removing the constraints of the HTTP/1.x protocol.
- It compresses headers. As these are often similar among a set of requests, this removes the duplication and overhead of data transmitted.
- It allows a server to populate data in a client cache through a mechanism called the server push.

### HTTP 3

The next major version of HTTP, HTTP/3 has the same semantics as earlier versions of HTTP but uses [QUIC](https://developer.mozilla.org/en-US/docs/Glossary/QUIC) instead of [TCP](https://developer.mozilla.org/en-US/docs/Glossary/TCP) for the transport layer portion. 

QUIC is designed to provide much lower latency for HTTP connections. Like HTTP/2, it is a multiplexed protocol, but HTTP/2 runs over a single TCP connection, so packet loss detection and retransmission handled at the TCP layer can block all streams. QUIC runs multiple streams over [UDP](https://developer.mozilla.org/en-US/docs/Glossary/UDP) and implements packet loss detection and retransmission independently for each stream, so that if an error occurs, only the stream with data in that packet is blocked.

## HTTP metode

**GET**

This method retrieves information from the given server using a given URI. GET request can retrieve the data. It cannot apply other effects on the data.

**HEAD**

The HEAD method is the same as the GET method. It is used to transfer the status line and header section only.

**POST**

The POST request sends the data to the server. For example, file upload, customer information, etc. using the HTML forms.

**PUT**

The PUT method is used to replace all the current representations of the target resource with the uploaded content.

**DELETE**

The DELETE method is used to remove all the current representations of the target resource, which is given by URI.

**CONNECT**

The CONNECT method is used to establish a tunnel to the server, which is identified by a given URI.



### Body in request

	HTTP requests have a body if they have a Content-Length or Transfer-Encoding header ([RFC 2616 4.3](http://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html)). If the 	   request has neither, it has no body, and your server should treat it as such.
	
	 PUT request does not require a body. A POST request sends data to the URL specified in the request. Unlike a PUT 	 request, a POST request does not have an optional body.
	
	According to Mozilla a DELETE request "may" have a body, compared to a PUT, which should have a body. By this it 	seems optional whether you want to provide a body for a DELETE request.



	PUT ->Should have a body
	
	DELETE ->Can have a body
	
	POST -> Must have a body

## Status Code

Odogovor sa servera  da li je uispesno nesto izvrseno.Ovo je broj od 3 cifre koji nosi poruku.

200-> OK requests

300-> Pokazuje da request ima vise mogucnosti za odgovor, indicates that the request has more than one possible responses

400->Client Error

500->Server Error

### Spring Rest

![image-20230130221125087](C:\Users\radoj\AppData\Roaming\Typora\typora-user-images\image-20230130221125087.png)

## Spring

Za svaku metodu imamo odgovarajucu anotaciju

```
GET -> @GetMapping
POST -> @PostMapping
.
.
.

```

@GetMapping(

name ->

)

#### Static Path

@GetMapping("/entities") 

#### Path Variable

@GetMapping("/entities/{id}") 

@GetMapping("/entities/{id}/test") 

{id} je placeholder za vrednost koja je dinamicka,i to moramo dodati preko anotacije

<span style="color:tomato">@PathVariable</span>

```
public void doSmeth(@PathVariable(value,name,required) tip naziv){

}
```

#### Query Path

@GetMapping("/entities")

put se pise ovako,ali mi moramo da dodamo te query parametre u funkciju koja je oznacena sa ovim

<span style="color:tomato">@RequestParam(value,name,required,defaultValue)</span>

```
public ovid doSmeth(@RequestParam(value,name,requered,defaultValue)){

}
```



MI cemo imati klasu koja je rest,i ta klasa nam je kontroler.Kontroler sadrzi endpointe,tj ta mapiranja i pored ovih @GET... mozemo u parametre da **inject** neke stvari

```
@RestController
class Rest{

	@GetMapping(...)
    public void doSmeth(...){

    }
}
```

<span style="color:tomato">@RequestHeader(name)</span>

```
	@GetMapping(...)
    public void doSmeth(@RequestHeader("name") String variableName){

    }
```

<span style="color:tomato">@DateTimeFormat (style,iso,pattern,fallbackPatterns)</span>

style-> default je "SS"

iso -> DateTimeFormat.ISO.NONE default

pattern -> to sami definisemo

fallbackPatterns -> ovo je ako primarni patern gore ne uspe

```
@GetMapping(...)
    public void doSmeth(@DateTimeFormat("dd-mm-yyyy") LocalDate variableName){

    }
```

<span style="color:tomato">@RequestPart(value,name,requered,default)</span>

Ovo je kada radimo sa multipart/form-data,tj sa fajlovima