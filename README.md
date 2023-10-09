# Mersal API - Docs

This is a documentation of the correct method to connect to Mersal as an external processing service.

## Required Methods

These methods need to be implemented by your service to be called by Mersal.

### 1. Order Binding

Implement a method to bind orders hosted by Mersal to be processed by your service.

```
POST /mersal/orders/bind

Accept: application/json
```

We will a send a request to this endpoint with the body containing multiple orders of the type `Order`

```csharp
public class Field
{
    public string Id { get; set; }
    public string FrindlyId { get; set; }
    public string? Value { get; set; }
    public DateTime UpdatedAt { get; set; }
}
public class Form
{
    public string FriendlyId { get; set; }
    public IEnumerable<Field> Fields { get; set; }
    public DateTime UpdatedAt { get; set; }
}
public class Order
{
    public string Id { get; set; }
    public string ServiceId { get; set; }
    public string? Title { get; set; }
    public string? AuthorId { get; set; }
    public IEnumerable<Form> Forms { get; set; }
    public OrderStatus Status { get; set; }
}
```

Your service is expected to store those orders locally and return an array of reference IDs in the form `OrderBind[]`

```csharp
public class OrderBind
{
    public string ReferenceId { get; set; }
    public string MersalId { get; set; }
}
```

### 2. Order Payment Confirmation

After an order has been paid successfully within Mersal's portal, your service will be notified by that using this method

```
POST /mersal/orders/{orderReferenceId}/confirm
```

Where `{orderReferenceId}` is the reference ID provided by the order binding method

## Order Management Method

- [ ] WIP
