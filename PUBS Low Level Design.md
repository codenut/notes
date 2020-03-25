{{toc depth="2" numbered="true"/}}

= Introduction =

Points Ux Backend Service(PUBS) is a package will provide a set of APIs for the [[My Points Page Refresh>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/]].

== System Overview ==

= Goals =

This document aims to explain the low-level technical design of PUBS which is one of the main components of the My Points Page Refresh. This aim is to identify the technical requirements, dependencies and implementation details needed on implementing **internal** APIs dedicated for the My Points Page Refresh.

= Scope =

PUBS is a package that will provide a set of APIs to provide customer Points related information. Information needed on the My Points Page such as customer name, ASIN name is outside the scope of PUBS. PURS should call the necessary APIs to get this information.

= Requirements =

== Functional Requirements ==

The APIs in PUBS should support the following front-end features:

=== Tabs ===

There will be 3 tabs:

* **All**: Default tab. Shows all Points transaction histories.
* **Points Earned: **Points that were added to the customer's account.
* **Points Spent/Cancelled: **Points that were deducted from the customer.

The number of results for each tab will be shown.

=== Filters ===

A year and month dropdown to limit the results.

=== Transactions ===

Each page will display 10 transactions.

Each transaction will display:

1. Date
1. Item name for intrinsic points/Campaign name for BBR points.
1*. A link to the Order Page for Intrinsic Points.
1. Amount of points for the transaction

Pending points should be hidden for transactions with:

* Points earned from completed purchases.
* Points issued from returned items.
* Points issued from canceled items.

=== Pagination ===

A previous and next buttons will be displayed in mobile.

A previous, next and page number buttons will be displayed in desktop. How many page number will displayed is not confirmed yet.

== Non-functional Requirements ==

=== Metrics ===

1. Availability:
1*. API Error Rate < 0.05%
1. TPS:
1*. 300 TPS in 1-min period for both (% class="inline-code" %)getPointsTransactions(%%)and(% class="inline-code" %)getPointsSummary(%%) APIs.
1. Transaction data:
1*. 99.999% of customers had less than 7900 transactions/year.
1*. 99.9% of customers had less than 1400 transactions/year.
1*. The top 5 customers had 306500, 443300, 546900, 1887600, 1927100 transactions/year
1**. These are mainly abusers.

=== Logs ===

Use CloudWatch/Timberlogs for logging.

=== Scalability ===

==== Caching ====

We will utilize a caching layer to reduce the number of calls to BTS for getting the customer's Points account information and transaction history.

Things to cache:

1. [[Transaction history>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TransactionHistoryFetch/]]
1. Number of total Points for each Transaction Types.

=== Tests ===

1. Unit Tests
1. Integration tests
1. UX tests
1. Load + Performance testing

= APIs =

The APIs in PUBS are listed below. The examples are representing the request and response in JSON format to make it more readable. Though in application, the request will be a Java object(SomeObjectRequest) that will be passed in the API method as a parameter and API method will return an object(SomeObjectResponse).

== getPointsSummary ==

This will be used to retrieve customer's Points related information.

=== Request ===

(% border="1" %)
|**Parameter**|**Type**|**Default Value**|**Description**
|customerId|String|Required|The value of the customer ID in string format.
|marketplaceId|String|Required|The value of the marketplace ID in string format.

=== Response ===

(% border="1" %)
|(% style="width:248px" %)**Field**|(% style="width:129px" %)**Type**|(% style="width:994px" %)**Description**
|(% style="width:248px" %)confirmedBalance|(% style="width:129px" %)Long|(% style="width:994px" %)The confirmed points in customer's account.
|(% style="width:248px" %)pendingBalance|(% style="width:129px" %)Long|(% style="width:994px" %)The pending points in customer's account.
|(% style="width:248px" %)expirationTimestamp|(% style="width:129px" %)Timestamp|(% style="width:994px" %)The timestamp when the confirmed points will expire

==== Errors ====

(% border="1" %)
|(% style="width:322px" %)**Name**|(% style="width:1048px" %)**Reason**
|(% style="width:322px" %)MissingRequiredParameterException|(% style="width:1048px" %)A missing required field was not found in the Request.

=== Example ===

==== Request ====

{{code language="json"}}
{
    "marketplaceId": "A1VC38T7YXB528",
    "customerId": "AJO6TERMKGJZN"
}
{{/code}}

==== Response ====

{{code language="json"}}
{
    "confirmedBalance": 125,
    "pendingBalance": 0,
    "expirationTimestamp": 1577804400
}
{{/code}}

=== Implementation ===

==== Response ====

The values in the response come from different APIs and below is how it will be retrieved.

===== confirmedBalance =====

Can be retrieved using PPAS [[getAccountSummary>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetAccountSummary]] API with customerId, marketplaceId as parameters.

===== pendingBalance =====

Can be retrieved using PPAS [[getAccountSummary>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetAccountSummary]] API with customerId, marketplaceId as parameters.

===== expirationTimestamp =====

Get the customer's latest activity from PCAS [[getCustomerLastActivity>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetCustomerLastActivity]] API with customerId, marketplaceId as parameters. Then add some arbitrary period to the latest activity.

== getPointsTransactions ==

This API returns the customer's Points transactions. The transactions in the response are sorted by timestamp in descending order.

=== Request ===

(% border="1" %)
|**Parameter**|**Type**|**Default Value**|**Description**
|customerId|String|Required|The customer ID of the customer that owns the Points transactions.
|marketplaceId|String|Required|The marketplace ID in string format.
|timestampRangeFilter|[[TimestampRange>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HTimestampRange]]|The timestamp range representation for the current month.|The timestamp range for the filter
|stateFilter|List[String]|null. No filter applied.|This is a list of [[states>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HStates]] to filter the transactions in the response.
|paging|[[Paging>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HPaging]]|See [[Paging>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HPaging]]|The pagination details for this request.

==== TimestampRange ====

(% border="1" %)
|(% style="width:209px" %)**Parameter**|(% style="width:196px" %)**Type**|(% style="width:427px" %)**Default Value**|(% style="width:539px" %)**Description**
|(% style="width:209px" %)start|(% style="width:196px" %)Timestamp|(% style="width:427px" %)The timestamp of the first day of the month.|(% style="width:539px" %)The timestamp of the starting datetime.
|(% style="width:209px" %)end|(% style="width:196px" %)Timestamp|(% style="width:427px" %)The timestamp of the last day of the month.|(% style="width:539px" %)The timestamp of the ending datetime.

==== Paging ====

(% border="1" style="height:195px; width:1119px" %)
|(% style="width:188px" %)**Parameter**|(% style="width:170px" %)**Type**|(% style="width:344px" %)**Default Value**|(% style="width:433px" %)**Description**
|(% style="width:188px" %)size|(% style="width:170px" %)Integer|(% style="width:344px" %)10|(% style="width:433px" %)The page size.
|(% style="width:188px" %)nextMarker|(% style="width:170px" %)String|(% style="width:344px" %)null|(% style="width:433px" %)(((
The marker to get the results of next page.
)))
|(% style="width:188px" %)previousMarker|(% style="width:170px" %)String|(% style="width:344px" %)null|(% style="width:433px" %)The marker to get the results of the previous page.
|(% style="width:188px" %)totalResults|(% style="width:170px" %)Integer|(% style="width:344px" %)0|(% style="width:433px" %)The total number of transactions for the request. This value is only populated in the response.

=== Response ===

(% border="1" %)
|(% style="width:205px" %)**Parameter**|(% style="width:197px" %)**Type**|(% style="width:969px" %)**Description**
|(% style="width:205px" %)paging|(% style="width:197px" %)[[Paging>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HPaging]]|(% style="width:969px" %)The paging for this response.
|(% style="width:205px" %)transactions|(% style="width:197px" %)List~[[[Transaction>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HTransaction]]]|(% style="width:969px" %)The list of customer transactions in this response.
|(% style="width:205px" %)transactionYears|(% style="width:197px" %)List[Integer]|(% style="width:969px" %)The list containing the years with transactions.

==== Transaction ====

(% border="1" %)
|(% style="width:204px" %)**Parameter**|(% style="width:202px" %)**Type**|(% style="width:965px" %)**Description**
|(% style="width:204px" %)id|(% style="width:202px" %)String|(% style="width:965px" %)The string to uniquely identify the transaction.
|(% style="width:204px" %)aggregationId|(% style="width:202px" %)String|(% style="width:965px" %)The string used to aggregate transactions(i.e. orderItemId, orderId, etc...)
|(% style="width:204px" %)timestamp|(% style="width:202px" %)Timestamp|(% style="width:965px" %)The timestamp when the transaction happened.
|(% style="width:204px" %)state|(% style="width:202px" %)String|(% style="width:965px" %)The [[state>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HStates]] of this transaction. CONFIRMED/PENDING/CANCELLED
|(% style="width:204px" %)type|(% style="width:202px" %)String|(% style="width:965px" %)The [[type>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HTypes]] of this transaction. Intrinsic/Credit Card/BBR
|(% style="width:204px" %)subtype|(% style="width:202px" %)String|(% style="width:965px" %)The [[subtype>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HSubtypes]] of this transaction.
|(% style="width:204px" %)amount|(% style="width:202px" %)Long|(% style="width:965px" %)The amount used for this transaction.
|(% style="width:204px" %)name|(% style="width:202px" %)String|(% style="width:965px" %)The name of this transaction.
|(% style="width:204px" %)useCaseDetails|(% style="width:202px" %)[[UseCaseDetails>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HUseCaseDetails]]|(% style="width:965px" %)A object that represents the object for this transaction.

==== UseCaseDetails ====

(% border="1" %)
|(% style="width:178px" %)**Type**|(% style="width:183px" %)**Value**|(% style="width:757px" %)**Description**
|(% style="width:178px" %)OrderItem|(% style="width:183px" %)[[OrderItem>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign#HOrderItem]]|(% style="width:757px" %)The order item for this transaction.

==== OrderItem ====

(% border="1" %)
|**Parameter**|**Type**|**Description**
|id|String|The order item ID.
|orderId|String|The order ID of the order item.
|asin|String|The asin of this order item.

==== Errors ====

(% border="1" %)
|(% style="width:316px" %)**Name**|(% style="width:1031px" %)**Reason**
|(% style="width:316px" %)MissingRequiredParameterException|(% style="width:1031px" %)A missing required field was not found in the Request.
|(% style="width:316px" %)InvalidTimestampRangeException|(% style="width:1031px" %)The timestampRangeFilter has invalid value. For example: start > end.

==== Types ====

(% border="1" %)
|**Name**|**Description**
|INTRINSIC|Transactions coming from for Retail, 3P, and Digital purchases.
|SPEND|Transactions for spending points.
|EARN|Transaction for earning points.
|BBR|Behaviour-based rewards.
|CREDIT_CARD|Transactions coming from using Amazon Credit Card on/off-Amazon purchases.
|PROMOTION|Transactions coming from promotions.
|EXPIRED|Transactions for expired points

==== Subtypes ====

(% border="1" %)
|**Name**|**Type**|**Description**
|PURCHASE|INTRINSIC|Transactions earned from purchases.
|REDEEMED|SPEND|Transactions spend on purchases.
|MANUAL_BBR|BBR|BBR transactions granted manually.
|AUTO_BBR|BBR|BBR transactions granted automatically.
|PAWS|PROMOTION|
|ESP|PROMOTION|Enterprise-seller points.
|MEP|PROMOTION|Member exclusive points.
|CBCC_ON_AMAZON|CREDIT_CARD|Transactions from CBCC on-Amazon.
|CBCC_OFF_AMAZON|CREDIT_CARD|Transactions from CBCC off-Amazon.
|REFUND_FROM_RETURN|EARN|Transactions for refunded Points from returned order.
|ADJUSTMENT_FROM_CUSTOMER_SERVICE|EARN|Transactions refunded by Customer Service.

==== States ====

(% border="1" %)
|**Value**|**Description**
|PENDING|This transaction represents Points about to be earned by customer
|CANCELLED_PENDING|This represents Pending Points transaction that was cancelled by the customer.
|CONFIRMED|This state is for Points that has been made available to the cusomter account.
|CANCELLED_CONFIRMED|This state is for Confirmed Points transactions that has been cancelled from the customer account.
|ENCUMBERED|This state is for Points transactions that were allocated to be spent by the customer.
|SPENT|The state for the confirmed encumbered Points transaction.
|REFUNDED|This transaction is for Points refunded to the customer's account.

=== Example ===

==== Request ====

{{code language="json"}}
{
    "marketplaceId": "A1VC38T7YXB528",
    "customerId": "AJO6TERMKGJZN",
    "stateFilters": ["PENDING"],
    "dateTimeRangeFilter": {
        "start": 1483196400,
        "end": 1577804400
    },
    "paging": {
        "size": 10,
        "nextMarker": "nextMarker",
        "previousMarker": "previousMarker"
    }
}
{{/code}}

==== Response ====

{{code language="json"}}
{
    "paging": {
        "size": 10,
        "nextMarker": "nextMarker",
        "previousMarker": "previousMarker",
        "totalResults": 24
    },
    "transactions": [{
        "id": "transactionId",
        "aggregateId": "aggregateId",
        "name": "transaction name",
        "timestamp": 1577804400,
        "amount": 125,
        "state": "PENDING",
        "type": "PROMOTION",
        "subtype": "PAWS",
        "useCaseDetails": {
            "__type": "OrderItem",
            "asin": "asin",
            "orderId": "orderId",
            "orderItemId": "orderItemId"
        }
    }],
    "transactionYears": [2018, 2019, 2020]
}
{{/code}}

=== Implementation ===

==== Response ====

The values in the response come from different APIs and below is how it will be retrieved.

===== transactions =====

The data for each transactions will be from the BTS' [[getInstrumentEvents>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetInstrumentEvents]] API. The API accepts an instrumentId that can be retrieved using a new [[getCustomerInstrumentId>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetCustomerInstrumentId]] API in PPAS.

===== transactionYears =====

The transactionYears is the list of years from the creation of customer's Points account to the current year. The account creation date time can be retrieved using BTS [[getInstrumentInformation>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentInformation/]] API.

= Data =

== Data Source ==

=== BTS ===

Also called as [[MonadService>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/]]. This service hosts the data for Points Transactions.

This the mapping of data type from BTS to Points Platform.

(% border="1" %)
|**BTS**|**Points Platform**
|(((
Account
)))|BTS provides an API to map between customerId and instrumentId as [[explained here>>https://w.amazon.com/bin/view/PointsPlatform/PointsBalance/#HIwanttoshowthecustomersbalanceonfrequentlyviewedpageorwidgetontheretailwebsite]]
|Instrument|Amazon Customer
|Events|Transactions

==== getInstrumentEvents ====

Transactions can be retrieved through the [[GetInstrumentEvents API>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentEvents/]]. Each transaction is saved as an event and composed of following BTS objects.

1. [[Event>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentEvents/#HEvent]]
1. [[Balance >>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentBalance/#Response]]
1. Amount

=== PCAS ===

===== getCustomerLastActivity =====

This is the API to get the customer's last activity which will be the basis of the Points expiration date.

= Dependencies =

== PACS - PointsAccountsService ==

Accessible from NAWS: No, we will need to onboard the service to allegiance

CloudAuth Support: Custom whitelist authorizer

== BTS - BalanceTrackingService ==

Accessible from NAWS: Yes

CloudAuth Support: Yes

=== Needed values ===

==== ASIN Name ====

The ASIN in the Transaction .OrderItem object can be retrieved using AAPI (% class="inline-code" %)[[GET marketplaces/marketplace-id/products/:asin/title>>url:https://api.corp.amazon.com/resources/marketplaces/:marketplace-id/products/:asin/title]](%%).

==== Customer Name ====

Can be retrieved using AAPI (% class="inline-code" %)[[GET marketplaces/:marketplace-id/customers/:customer-id/profile/public>>url:https://api.corp.amazon.com/operations/Se9myegbT6/miniProfile]](%%)with customerId as a parameter.

=== Key requirements: ===

1. Compute the the operation/second or ASIN/second.
1*. Expected max TPS for getPointsTransactions API is [[367 TPS in 1-min period>>https://tiny.amazon.com/13hly3c40/iGraph]]
1*. 7 TPS/second * 10(results per request) * 2(PD 2020 peak multiplier) = 140 OPS
1*. for get customer public name = 100 OPS / 10(results per request) = 14 OPS
1*. Total = **154 OPS**
1*. As per this [[quip>>https://quip-amazon.com/W1EbA1KO9E1R/AmazonAPI-ProductV2-New-Traffic-Onboarding]] document, we do not need a new hardware.

Follow the on-boarding steps [[here>>https://evergreen.corp.amazon.com/amazon-api/onboarding/Data-Consumer-Onboarding/Getting-Started#onboard-with-amazonapi]].

Accessible from NAWS? Yes

CloudAuth Support: Not until April 30. We can use AAA in EC2 as an alternative.

= Risks =

=== There is an ongoing [[discussion>>https://sim.amazon.com/issues/PointsIntake-509]] on choosing PUBS or PAcS as a backend service for My Points Page. What happens when PAcS is chosen? Is this design still relevant? ===

Partly, yes. If PAcS is chosen as the backend service, then PUBS will be implemented as a package and these are the things that are still relevant:

1. API Design - The API will still be the same.
1*. PAcS will expose an API which is similar to PPAS's getCustomerTransactions.
1*. PUBS will consume the APIs from PAcS.
1. Scalability - This still stands, since the load will still be the same.
1. Metrics - Same with item 2.
1. Dependencies - Dependencies remain the same.

And these are not:

1. Infrastructure - PAcS is in MAWS.
11. PAcS needs to be onboarded to Allegiance.
11. PAcS need to be onboarded to CloudAuth and an Authorizer should be added to existing APIs.
11*. This will need Infosec approval.
11. Caching - The proposed solution will be using Redis in ElasticCache. We need to setup a Redis server in MAWS in an Apollo Environment or implement the caching in PURS with ElasticCache.

= Open Questions =

1. Should we put the transactionYears in the [[getPointsTransactions>>https://w.amazon.com/bin/view/JPMobile/Development/PointsCX/MyPointsPage/TechDesign/PUBSLowLevelDesign/#HgetPointsTransactions]] API? How to come up with the list of transactionYears? It is [[currently constructed>>https://code.amazon.com/packages/PointsPlatformAccountsService/blobs/e16e9f761e53dcb8ff510baf1596b520d1a490a8/--/src/com/amazon/pointsplatform/service/util/TransactionYearsUtil.java#L31]] by generating a list of years from the account creation year to the current year.

= FAQs =

= Related Links =

[[Creating EC2 instance in NAWS>>https://w.amazon.com/bin/view/DEX/JP/Process/NAWS_EC2/]]

[[BTS - GetInstrumentBalance>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentBalance/]]

[[BTS - GetInstrumentEvents>>https://w.amazon.com/bin/view/StoredValuePlatform/Development/MonadService/APIs/V20120101/GetInstrumentEvents/]]
