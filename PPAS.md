# PPAS

## APIs
### creditPoints
#### Request
```json
{
    "Operation": "com.amazon.pointsplatform.model#creditPoints",
    "Service": "com.amazon.pointsplatform.model#PointsPlatformAccountsService",
    "Input": {
        "transactionDetail": {
            "pointsContext": {
                "amount": 20000,
                "creditType": "BUY",
                "pointsOfferIdentifier": 6
            },
            "transactionState": "CONFIRM_AND_CLOSE",
            "externalReferenceKey": "orderExternalReferenceKeyValid/testInitializeCancelRequest/4",
            "subKeyExtension": "shipmentId/shipmentItemId/4",
            "subKey": "campaignId4",
            "transactionAttributes": "e0J1c2luZXNzRXZlbnREYXRlOjE1MDczNzI5NTQ4MDksQ2FtcGFpZ25JZDoiMiIsSW50ZXJuYWxDb21tZW50OiJuYW1lPXVzZXJuYW1lLHZhbHVlPXBvaW50c3lzLHR5cGU9c3RyO25hbWU9cmVhc29uLHZhbHVlPWNycC1jYmNjLWNvbmZpcm1lZCx0eXBlPXN0cklkIixNYXJrZXRwbGFjZUlkOjYsRXZlbnRUeG5UeXBlOiJQQVlNRU5UX0lOU1RSVU1FTlRfR1JBTlQiLFBheW1lbnRUcmFuc2FjdGlvbklkOiI3MDEwMDc5NjM0Njg1IixQYXltZW50VHJhbnNhY3Rpb25UYWc6Ijk0MS03OTM1NTQ3LTY0OTU4NDEifQ=="
        },
        "customerId": "A1X9ZVX8L9DG6F",
        "session": {
            "idempotencyKey": "A1X9ZVX8L9DG6F_192eeeee0223030234e32de343"
        },
        "applicationContext": {
            "appName": "JPPoints",
            "pointsProgramName": "jp-points",
            "marketPlaceId": "A1VC38T7YXB528"
        }
    }
}
```

### getCustomerTransactions
#### Request
```json
{
    "Operation": "com.amazon.pointsplatform.model#getCustomerTransactions",
    "Service": "com.amazon.pointsplatform.model#PointsPlatformAccountsService",
    "Input": {
        "customerId": "A1X9ZVX8L9DG6F",
        "marketplaceId": "A1VC38T7YXB528",
        "fromTime": 1577804400,
        "toTime": 1601478000,
        "batchSize": 100,
        "batchNumber": 1,
        "applicationContext": {
            "appName": "JPPoints",
            "pointsProgramName": "jp-points",
            "marketPlaceId": "A1VC38T7YXB528"
        }
    }
}
```

## Event Info
```json
{
    "balanceChange":
    {
        "currencyBalanceAvailable":
        {
            "amount": "0.000000",
            "currencyCode": "_AP"
        },
        "pointsBalanceAvailable": 0,
        "pointsBalancePendingIn": 40,
        "pointsBalancePendingOut": 0,
        "pointsExpiryBalanceAvailable": 0,
        "pointsNonExpiryBalanceAvailable": 0
    },
    "eventContextAttributes": "e0J1c2luZXNzRXZlbnREYXRlOjE1ODQ5MzMxMjMwMDAsU2t1OiJQUDEwMDAwNTUxIixBc2luOiJQUDEwMDAwNTUxIixDYW1wYWlnbklkOiIxIixHTFByb2R1Y3RHcm91cElkOjEwNyxNYXJrZXRwbGFjZUlkOjYsTWVyY2hhbnRJZDoxMixPcmRlcklkOiI5NDEtOTk2OTE4MS0zMzM3NDQ0IixPcmRlckl0ZW1JZDo2MTc1NjMyMzg0NjQ1LFByaWNlOjEwMDAuLFF1YW50aXR5OjQsRXZlbnRUeG5UeXBlOiJPUkRSIixWZXJzaW9uOiIyLjAifQ==",
    "eventDate": 1.58493320981E9,
    "expiryInDays": 0,
    "numberOfPoints": 0,
    "quantity": 0,
    "runningBalance":
    {
        "currencyBalanceAvailable":
        {
            "amount": "17.000000",
            "currencyCode": "_AP"
        },
        "pointsBalanceAvailable": 17,
        "pointsBalancePendingIn": 110,
        "pointsBalancePendingOut": 3,
        "pointsExpiryBalanceAvailable": 0,
        "pointsNonExpiryBalanceAvailable": 0
    },
    "transactionAmount":
    {
        "amount": "40.000000",
        "currencyCode": "_AP"
    }
}
```

# Transaction

| PUBS           | PPAS                           | Exists? |
|----------------|--------------------------------|---------|
| id             | eventId                        | NO      |
| aggregationId  | eventContextAttributes.OrderId | yes     |
| timestamp      | eventDate                      | yes     |
| state          |                                |         |
| type           |                                |         |
| subtype        |                                |         |
| amount         | transactionAmount.amount       | yes     |
| name           |                                |         |
| useCaseDetails |                                |         |

## useCaseDetails
### OrderItem
| PUBS    | PPAS                               | Exists? |
|---------|------------------------------------|---------|
| id      | eventContextAttributes.OrderItemId | yes     |
| orderId | eventContextAttributes.OrderId     | yes     |
| asin    | eventContextAttributes.Asin        | yes     |
