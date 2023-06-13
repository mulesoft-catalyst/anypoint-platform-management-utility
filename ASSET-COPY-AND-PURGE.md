# Asset Copy & Purge

The Asset Copy & Purge endpoints allows moving assets from one Business Group/ Environment to another with initially copying them to the target business group and once they have been verified to be correctly copied they can be purged from the source business group. The API currently supports the move of the following assets:

- **Anypoint MQ** - Queues - both standard and FIFO, exchanges and client applications

> **NOTE:** `The client applications which are copied to the target business group/ environment will have a new client id/ clientsecret and they need to be updated in the application properties`

The asset move is usually a 2-step process -

_Step 1_ : Copy assets to the target business group and environment and leave the source asset in its current state. This step is performed by invoking the following endpoint

```
POST /management-plane/assets/copy
```

_Step 2_ : Purge assets from the target business group and environment once they are being used by the migrated applications. This step is performed by invoking the following endpoint

```
POST /management-plane/assets/purge
```

Both endpoints above needs to be invoked with the following information (described in further detail below):

- From Business Group
- To Business Group
- Environment Specific Information
    - From Environment
    - To Environment
    - Anypoint MQ (Optional)
        - Region
        - Destinations (Optional)
        - Clients (Optional)

## Anypoint MQ

These assets can be moved at the same time with the dependency of assets - such as DLQ and Exchange Bindings are handled within the API. Once successfully copied and verified the assets can be purged

The _Anypoing MQ Destination Asset Name_ is the name of the destinations as below

![image](/images/img-acp1.png)

and the _Anypoing MQ Client Asset Name_ is the name of the clienets as below

![image](/images/img-acp2.png)

For eg:

```
curl --location --request POST 'http://localhost:8081/api/management-plane/assets/copy' \
--header 'Authorization: Bearer xxxxxxxxxxxxxxx' \
--header 'Content-Type: application/json' \
-data-raw '{
    "fromBusinessGroup": "MuleSoft",
    "toBusinessGroup": "C4E",
    "environmentSpecificAssets": {
        "fromEnvironment": "Development",
        "toEnvironment": "Sandbox",
        "anypointMQ": {
            "region": "eu-west-1 ",
            "destinations": [
                "asset-migration-test-q1",
                "asset-migration-test-fq1",
                "asset-migration-test-ex1",
                "asset-migration-test-fq1-dlq",
                "asset-migration-test-q 1-dlq",
                "asset-migration-test-q 1-dlq1",
                "asset-migration-test-q1-dlq2"
            ],
            "clients": [
                "asset-migration-test-client1",
                "asset-migration-test-client2"
            ]
        }
    }
}
```

> **NOTE:**` The above request will copy the asset exactly with the same details from the source to the target business group. The order of destinations is not important but all queues such as DLQs and Exchange Bindings must be present in the the same request or already existing in the target business group`
