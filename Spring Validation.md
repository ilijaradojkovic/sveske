# Spring Validation

Ovo je biblioteka koju uvozimo kako bi validirali parametre ulaznih objekata koji dolaze na API u body delu ili u requestu

<span style="color:#ff6347">@NotNull</span>

<span style="color:#ff6347">@Positive</span>

<span style="color:#ff6347">@NotBlank</span>

<span style="color:#ff6347">@Min</span>

<span style="color:#ff6347">@Max</span>

<span style="color:#ff6347">@Email</span>

<span style="color:#ff6347">@NotEmpty</span> -> Stavimo na listu 

<span style="color:#ff6347">@Pattern(pattern)</span>

<span style="color:#ff6347">@AssertFalse </span>->Na boolean anotaciju

<span style="color:#ff6347">@AssertTrue ->Na boolean anotaciju</span> 

<span style="color:#ff6347">@NegativeOrZero</span>

<span style="color:#ff6347">@Null </span>

<span style="color:#ff6347">@PositiveOrZero</span>

<span style="color:#ff6347">@Negative</span>

<span style="color:#ff6347">@Size</span>

<span style="color:#ff6347">@Future</span> ->Date in future

<span style="color:#ff6347">@Past </span> ->Date in past





Pored ovih predefinisanih mi mozemo da pravimo i nase custom validacije



Prvo cemo definisati custom anotaciju

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE, ElementType.RECORD_COMPONENT, ElementType.PARAMETER, ElementType.FIELD})
@Constraint(validatedBy = {NoDuplicateAddonValidator.class})
public @interface NoDuplicateAddon {
    String message() default "Elements  addons cant be duplicate";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

Jako je bitno da ona ima ova 3 elementa,jer su ona default kako bi se prepoznala od strane Spring Validaiton bibliokete.

Takodje da bi radila korektno moramo da stavimo ove element type da bude odgovarajuci(Kada sam radio primer sa listom nije htelo da validira dok niam stavio ElementType.PARAMETER)

Kada ovo napravimo najbitnija stvar je da stavimo @Constraint(validator)-> ovo je anotacija iz Spring Validation bibliokete koja kaze ok imas anotaciju koja je za validaciju ,ali mi daj objekat koji to validira.To je ovaj objelat validator on handle tu validaciju.

Znaci moramo da napravimo klasu koja ce to da handle

```java
public class NoDuplicateAddonValidator implements ConstraintValidator<NoDuplicateAddon, List<AddonPricesPerIntervalRequest>> {
    @Override
    public boolean isValid(List<AddonPricesPerIntervalRequest> addonPricesPerIntervalRequests, ConstraintValidatorContext constraintValidatorContext) {
        Map<Long, String> pricesPerIntervalDuplicate = new HashMap<>();
        addonPricesPerIntervalRequests.forEach(addonPricesPerIntervalRequest ->
                pricesPerIntervalDuplicate.computeIfAbsent(addonPricesPerIntervalRequest.addon(), k -> addonPricesPerIntervalRequest.addon() + "")

        );
        return pricesPerIntervalDuplicate.size() == addonPricesPerIntervalRequests.size();

    }
}
```

ConstraintValidator<Anotacija,InputParam> je interfejs koji validira u zavisnosti gde mi anotaciju stavimo u kodu.

bitno je ova logika u isValid jer ako vraca true onda nema greske o validaciji,ali ako vraca false onda ima greske.



@Valid->da bi ove anotacije radile moramo da stavimo @Valid kako bi spring ih validirao

```java

@PostMapping(PROPERTY_URL_CREATE)
public Mono<ExternalApiSuccessResponse> create(@RequestBody @Valid PropertyCreationRequest propertyCreationRequest) {
...
}
```

Vidimo da imamo @Valid odmah pored objekta koji primamo,tako cemo mu dati do znanja da treba da primeni validatore.

```java
public record PropertyCreationRequest(
        @NotNull
        @NotBlank
        String name,

        Boolean automaticApproval,

        @NotNull
        PropertyType propertyType,

        @Positive
        Integer maxPersons,

){
}
```



Kako imamo prost ovjekat sa basic poljima ovo valid ce da dvalidira sve bez problema.

Ali ako imamo malo komplikovaniji nested objects onda moramo da dodamo @Valid anotaciju ispred njih ponovo kako bi se nested objekti opet validirali

```java

public record PropertyCreationRequest(
        @NotNull
        @NotBlank
        String name,
    
        Boolean automaticApproval,
    
        @NotNull
        @Valid
        PropertyAddressRequest address,

        @NotNull
        @NotEmpty
        @NoDuplicateInterval
        List<@Valid PropertyPricesPerInterval> pricesPerInterval,

        @NotNull
        @NotEmpty
        @NoDuplicateAddon
        List<@Valid AddonPricesPerIntervalRequest> addons,

) {

}
```

VIdimo da imamo komplikovan objekat PropertyAddressRequest koji zelimo da validiramo zajedno sa celim objektom PropertyCreationRequest zato na njemu u tom objektu mora da stoji @Valid jer je nested i za validiranje nested objekata opet mora da stoji @Valid na njima.

Isto tako i za listu ako zelimo posebno da validiramo sveki objekat liste.

Automatski custom exception vraca ConstraintNotValidException.class