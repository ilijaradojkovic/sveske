# XML

XML parsiranje se radi na 2 nacina:

1)DOM

2)SAX



## DOM

DOM je document parsiranje,on radi sve odjednom,ucitava ceo xml fajl odmah odjednom,to moze nekad biti naporno pa nije prepoljucljivo u nekim situacijama.

Glavne klase:

- <span style="color:red">DocumentBuilderFactory</span>
- <span style="color:red">DocumentBuilder</span>
- <span style="color:red">Document</span>
- <span style="color:red">Node</span>

```java
DocumentBuilderFactory documentBuilderFactory= DocumentBuilderFactory.newInstance();
            DocumentBuilder builder = documentBuilderFactory.newDocumentBuilder();
            Document parse = builder.parse(new File(""));

            parse.normalizeDocument(); -> normalizuje dokument ako ima neke nepravilnosti,radi dodatnu optimizaicju
            parse.getParentNode()->normalizuje dokument ako ima neke nepravilnosti
            parse.getElementsByTagName() -> uzima na osnovu taga
            parse.getElementById() -> uzima na osnovu atributa id koji je na tagu,id je jedinstven za sve
```





