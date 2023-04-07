# Stripe

Da bi bilo sta radili sa Stripe-om moramo da uzmemo private key i pre svakog poziva metode da nalepimo

```java
Stripe.apiKey=NasSecretKey 
```

kako bi metoda radila.

## Customers

#####  Create

Koristimo <span style="color:#EE4B2B">Customer.create</span>  staticku metodu

```java
@PostMapping
public String createCustomer(@RequestBody MyCustomer myCustomer) throws StripeException {

    Stripe.apiKey= stripeKey;
    Map<String, Object> params = new HashMap<>();
    params.put("description",myCustomer.description());
    params.put("email", myCustomer.email());
    params.put("name", myCustomer.name());
    params.put("phone", myCustomer.phoneNumber());
    params.put("balance", myCustomer.balance());

    return Customer.create(params).toJson();
}
```



##### Get customer by id

Koristimo <span style="color:#EE4B2B">Customer.retrive</span>  staticku metodu

```java
@GetMapping("/{id}")
public String getCustomer(@PathVariable("id") String id) throws StripeException {
    Stripe.apiKey=stripeKey;

  return  Customer.retrieve(id).toJson();
}
```



##### Delete customer

Koristimo <span style="color:#EE4B2B">customerobject.delete()</span>  staticku metodu

```java
@DeleteMapping("/{id}")
public String deleteCustomer(@PathVariable("id") String id) throws StripeException {
    Stripe.apiKey=stripeKey;
    Customer customer = Customer.retrieve(id);
    return customer.delete().toJson();
}
```



##### Update customer

Koristimo <span style="color:#EE4B2B">customerobject.update(params)</span>  staticku metodu

```java
@PutMapping("/{id}")
public String updateCustomer(@PathVariable("id") String id,@RequestBody MyCustomer myCustomer) throws StripeException {
    Stripe.apiKey=stripeKey;
    Customer customer=Customer.retrieve(id);
    Map<String, Object> params = new HashMap<>();
    params.put("description",myCustomer.description());
    params.put("email", myCustomer.email());
    params.put("name", myCustomer.name());
    params.put("phone", myCustomer.phoneNumber());
    params.put("balance", myCustomer.balance());
    return customer.update(params).toJson();
}
```



###### Get all customers

Koristimo <span style="color:#EE4B2B">Customer.list(params)</span>  staticku metodu

```
@GetMapping()
public String getAllCustomers(@RequestParam("limit") int limit) throws StripeException {
    Stripe.apiKey=stripeKey;
    Map<String, Object> params = new HashMap<>();
    params.put("limit",limit);
    return Customer.list(params).toJson();
}
```





###### Search customers

Koristimo <span style="color:#EE4B2B">Customer.search(params)</span>  staticku metodu

ali ovo params su sada <span style="color:#EE4B2B">CustomerSearchParams</span> gde mi build custom quiery za search

```
@GetMapping("/search")
public String searchCustomers(@RequestParam("name") String name) throws StripeException {
    Stripe.apiKey=stripeKey;
    CustomerSearchParams params =
            CustomerSearchParams
                    .builder()
                    //need custom string to be build "name:'fakename' AND metadata['foo']:'bar'
                    .setQuery("name:'"+name+"'")
                    .build();

    CustomerSearchResult result = Customer.search(params);

    return result.toJson();
}
```

# NJIHOV API JE DOBRO OBJASNJEN NEMA POTREBE PISATI OVDE