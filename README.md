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

## Order Management Worlflow

The workflow of managing the synchronization of orders between Mersal and an external service is as follows

1. **Order Binding** (M) - Mersal sends a batch of pending submitted orders to the external service using `POST /mersal/orders/bind` every 10 ~ 15 minutes, the service stores the orders locally and binds a unique ID to each one, sending that ID back along with Mersal's ID back as the response.

2. **Order Custom Notification** (S) - The external service sends a custom notification to the order's author (customer) using `POST /api/orders/{orderMersalId}/message` informing them to take specific action for the order to be accepted (personal attendance, tax payment, etc).

3. **Order Acceptance/Rejection** (S) - The external service sends a notification to Mersal using `POST /api/orders/{orderMersalId}/accept` when an order has been validated and accepted, or `POST /api/orders/{orderMersalId}/reject` when an order has been rejected.

4. **Order Confirmation** (M) - Mersal sends a notification to the external service using `POST /mersal/orders/{orderReferenceId}/confirm` indicating that an order payment has been completed and is ready to be fulfilled with the order's items.

5. **Order Fulfillment** (S) - The external service informs Mersal using `POST /api/orders/{orderMersalId}/fulfill` that the order has been fulfilled and is ready for pickup by the customer, the service can optionally send a digital copy of the resulted documents.

## API Documentation

### Authentication

Mersal API uses JWT authentication (Bearer scheme), by injecting a token in the `Authorization` header

```
Authorization: Bearer {token}
```

The token can be acquired through Mersal Portal

## Message Order

#### Endpoint

```
POST /api/orders/{orderMersalId}/message
```

#### Route Parameters

| Parameter Name  | Type     | Description                                      |
| --------------- | -------- | ------------------------------------------------ |
| `orderMersalId` | `String` | The ID provided by Mersal for a specific `Order` |

#### Request Body (application/json)

| Property Name | Type     | Description                                        |
| ------------- | -------- | -------------------------------------------------- |
| `message`     | `String` | The message that will be displayed to the customer |

#### Responses

| Status Code        | Body Type             | Description                                                |
| ------------------ | --------------------- | ---------------------------------------------------------- |
| 200 (OK)           | N/A                   | The message has successfully been submitted to the `Order` |
| 401 (UNAUTHORIZED) | [ApiError](#apierror) | Invalid token                                              |
| 404 (NOT FOUND)    | [ApiError](#apierror) | Given ID does not match any `Order`                        |

## Accept Order

#### Endpoint

```
POST /api/orders/{orderMersalId}/accept
```

#### Route Parameters

| Parameter Name  | Type     | Description                                      |
| --------------- | -------- | ------------------------------------------------ |
| `orderMersalId` | `String` | The ID provided by Mersal for a specific `Order` |

#### Request Body (N/A)

N/A

#### Responses

| Status Code        | Body Type             | Description                                                           |
| ------------------ | --------------------- | --------------------------------------------------------------------- |
| 200 (OK)           | N/A                   | The `Order` has successfully been accepted and is now pending payment |
| 401 (UNAUTHORIZED) | [ApiError](#apierror) | Invalid token                                                         |
| 404 (NOT FOUND)    | [ApiError](#apierror) | Given ID does not match any `Order`                                   |

## Reject Order

#### Endpoint

```
POST /api/orders/{orderMersalId}/reject
```

#### Route Parameters

| Parameter Name  | Type     | Description                                      |
| --------------- | -------- | ------------------------------------------------ |
| `orderMersalId` | `String` | The ID provided by Mersal for a specific `Order` |

#### Request Body (application/json)

| Property Name | Type     | Description                                                          |
| ------------- | -------- | -------------------------------------------------------------------- |
| `reason`      | `String` | A message explaining to the customer the reason behind the rejection |

#### Responses

| Status Code        | Body Type             | Description                                                             |
| ------------------ | --------------------- | ----------------------------------------------------------------------- |
| 200 (OK)           | N/A                   | The `Order` has successfully been rejected and is now back into `Draft` |
| 401 (UNAUTHORIZED) | [ApiError](#apierror) | Invalid token                                                           |
| 404 (NOT FOUND)    | [ApiError](#apierror) | Given ID does not match any `Order`                                     |

## Fulfill Order

#### Endpoint

```
POST /api/orders/{orderMersalId}/fulfill
```

#### Route Parameters

| Parameter Name  | Type     | Description                                      |
| --------------- | -------- | ------------------------------------------------ |
| `orderMersalId` | `String` | The ID provided by Mersal for a specific `Order` |

#### Request Body (application/json)

| Property Name | Type       | Description                                                                                                                     |
| ------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `mediaFiles`  | `String[]` | _optional_ - An array of file URLs that are hosted by either Mersal or any third party service (**must be publicly available**) |

#### Responses

| Status Code        | Body Type             | Description                                                                                                                                                                     |
| ------------------ | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200 (OK)           | N/A                   | The `Order` has successfully been fulfilled and is now ready for pickup by the customer (**and documents attached in the body request can be viewed by the customer directly**) |
| 401 (UNAUTHORIZED) | [ApiError](#apierror) | Invalid token                                                                                                                                                                   |
| 404 (NOT FOUND)    | [ApiError](#apierror) | Given ID does not match any `Order`                                                                                                                                             |

### Data Types

#### ApiError

```csharp
public class ApiError
{
    public string Code { get; set; };
    public string Message { get; set; };
}
```
