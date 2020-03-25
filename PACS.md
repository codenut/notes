# PACS

## APIs

### getCustomerTransactions
This is a new API in PACS for the new My Points Page. The previous version of this API is specific to the current My Points Page. The response for this new API will be more flexible that will be usable for different kinds of clients.

The request and response for this API is similar to PPAS getCustomerTransactions API.

#### AWS Onboarding
PACS is currently not accessible to NAWS and it is also not CloudAuth enabled. The reason why it is not onboarded to CloudAuth is that one of it's clients is a Gurupage page and Gurupa does not support CloudAuth.

Here are the steps to make it accessible to NAWS:
1. Onboard to Allegiance.
2. Create a [whitelist authorizer](https://code.amazon.com/packages/AlexaSkillLatencyService/blobs/b749b3545d2a3b44a879e9825ff502cc06303728/--/src/com/amazon/alexaskilllatencyservice/CoralModule.java#L193) for current APIs.
    * Related [sage](https://sage.amazon.com/questions/636317) question.
4. Onboard the new API to CloudAuth.
