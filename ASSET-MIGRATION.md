# Asset Migration

The Asset Migration endpoints allows the migration of assets from one Business Group/ Environment to another. The API currently supports the migration of the following assets:

- **Design Center** - API Specification Projects with only the \_master \_branch with RAML, RAML Fragments, Exchange Dependencies, JSON/ XML examples, Markdown (.md) files etc
  
- **Exchange** - REST API type Exchange Assets with multiple versions and documentation pages with the same formatting etc.
> NOTE: `Images uploaded to any of the pages, is currently not included in the migration and needs to be upoaded manually to new Business Group`

- **API Manager** - One or more API Instances from API Manager for a single asset including all Policies, Contracts, SLA Tiers
> NOTE: `API Alerts are not included in the migration`

- **Runtime Manager (CloudHub & Runtime Fabric)** - Mule applications deployed to a CloudHub (including applications deployed with Static IP) and Runtime Fabric. There will be no change to the clients accessing the application because the endpoint will remain the same after the migration
> NOTE: `Migration of Runtime Manager assets will result in all Object Store entries to be reset. Also, Runtime Manager Alerts are not included in the migration and the application may have a downtime in the environment (see DLB section below) where the app is being migrated.`

The asset migration is usually a 2-step process -

_Step 1_ : Migrate assets to the target business group and environment and mark assets in the source business group and environment as "unuseable". This step is performed by invoking the following endpoint

```
POST /management-plane/assets/migrate
```

_Step 2a_ : Commit the migrated assets to the target business group and environment and deletes all assets in the source business group and environment. This step is performed by invoking the following endpoint

```
POST /management-plane/assets/migrate/commit
```

OR

_Step 2b_ : Rollback the migrated assets by deleting them from the target business group and environment and reinstate all assets in the source business group and environment to their original state. This step is performed by invoking the following endpoint

```
POST /management-plane/assets/migrate/rollback
```

All of the 3 endpoints above needs to be invoked with the following information (described in further detail below):

- From Business Group
- To Business Group
- Design Center Asset Name (Optional)
- Exchange Asset Name (Optional)
- Asset Minor Version Override (Optional)
- Environment Specific Information
    - From Environment
    - To Environment
    - API Manager Asset Name (Optional)
    - CloudHub Asset Name (Optional)
    - RTF Asset Name (Optional)
    - Delete Asset (Optional)

The asset migration endpoint can be invoked with all the asset names at the same time or specific asset names from each component (for eg: Design Center, Exchange etc) is required. The recommendation would be to use the following steps as described below

#### Design Center, Exchange and API Manager

These assets can be migrated at the same time as the dependency of assets between these components are handled within the API. Once migrated the assets are marked as deprecated (Design Center doesn't allow deprecation of the project so they are renamed to indicate that they have been migrated).

##### Design Center
The _Design Center Asset Name_ is the name of the project as below

![image](/images/img-am1.png)

Once migrated the project is renamed

![image](/images/img-am2.png)

##### Exchange

The _Exchange Asset Name_ is the asset id which can be obtained from the URL as below

![image](/images/img-am3.png)

Once migrated the asset is deprecated

![image](/images/img-am4.png)

##### API Manager

The _API Manager Asset Name_ is the **same** asset id from Exchange as above which may not exactly match with the name of the API instance in the environment as below

![image](/images/img-am5.png)

Once migrated the instance is deprecated

![image](/images/img-am6.png)

For eg:

```
curl --location --request POST 'http://localhost:8081/api/management-plane/assets/migrate' \
--header 'Authorization: Bearer xxxxxxxxxxxxxxx' \
--header 'Content-Type: application/json' \
-data-raw '{
    "fromBusinessGroup": "MuleSoft",
    "toBusinessGroup": "C4E",
    "designCenterAssetName": "asset-migration-test-api",
    "exchangeAssetName": "asset-migration-test-api",
    "environmentSpecificAssets": {
        "fromEnvironment": "Development",
        "toEnvironment": "Sandbox",
        "apiManagerAssetName": "asset-migration-test-api"
    }
}'
```

> `NOTE: The above request will migrate the asset exactly with the same details from the source to the target business group. In certain circumstances, if the asset was already previously migrated to the target business group and deleted after more than 7 days which resulted in a soft-delete, the same name and version cannot be re-used and the following error is thrown`

![image](/images/img-am7.png)

> Adding the following field to the request will allow the assets to override the minor version with whatever has been provided in the input before importing to Design Center (exchange.json file), Exchange and API Manager (API instance asset version).

```
"assetMinorVersionOverride": "1.1"
```

#### Runtime Manager

These assets are **recommended** to be migrated separately after the related Design Center, Exchange and API Manager assets have been migrated to the target business group. This is to mainly to ensure that the APIs using Auto Discovery, will need the new API Instance Ids to be passed as a property for Runtime Manager deployments for both CloudHub and RTF.

**Additionally, any properties which are securely hidden by adding as** secureProperties **in the mule_artifact.json file will also needs to be added in the asset migration post request because those values are encrypted and is not accessible by this API **

For eg, the following properties will be replaced while deploying the application from Source to Target Business Groups. Also, any new properties added to the "properties" element will be included as part of the deployment

```
curl --location --request POST 'http://localhost:8081/api/management-plane/assets/migrate' \
--header 'Authorization: Bearer xxxxxxxxxxx' \
--header 'Content-Type: application/json' \
--data-raw '{
    "fromBusinessGroup": "MuleSoft",
    "toBusinessGroup": "C4E",
    "environmentSpecificAssets": {
        "fromEnvironment": "Development",
        "toEnvironment": "Sandbox",
        "runtimeManager": {
            "cloudHubAssetName": "exp-asset-migration-test-api",
            "properties": {
                "anypoint.platform.client_id": "xxxx",
                "anypoint.platform.client_secret": "xxxx",
                "api.id": "xxxx"
            }
        }
    }
}
```

.This request also follows the 2-step migration process.

- Step 1 stops the "Started" (CloudHub)/ "Running" (RTF) application from the source business group and redeploys the application by appending "-m" suffix to the original name to the target business group
- If Step 2a (Commit) is issued, it deletes the "Undeployed" (CloudHub)/ "Not Running" (RTF) application from the source business group and redeploys the application with "-m" suffix with its original name (without the suffix) and stops the application with the "-m" suffix on the target business group and environment.
- If Step 2b (Rollback) is issued, it starts the "Undeployed" (CloudHub)/ "Not Running" (RTF) application in the source business group and deletes the application with "-m" suffix from the target business group and environment.

This migration process allows to retain the Anypoint Monitoring logs and metrics data for the application in the Source business group for a specified period of time (until Step 2 is performed) to investigate any issues/ problems

> NOTE: If retaining the above application logs/ metrics is not required adding the following field to the request will perform a 1-step migration and will skip the Step 2 completely

```
{
    "fromBusinessGroup": "MuleSoft",
    "toBusinessGroup": "C4E",
    "environmentSpecificAssets": {
        "fromEnvironment": "Development",
        "toEnvironment": "Sandbox",
        "runtimeManager": {
            "deleteAsset": true,
            "cloudHubAssetName": "exp-asset-migration-test-api",
            "properties": {
                "anypoint.platform.client_id": "xxxx",
                "anypoint.platform.client_secret": "xxxx",
                "api.id": "xxxx"
            }
        }
    }
}
```

> In this case, the application from the source business group is deleted first and deployed to the target business group with the same name but this approach would mean that **all application logs/ metrics data from the source application will be deleted** and will not accessible anymore

##### CloudHub

> NOTE: For Runtime Manager deployments please ensure that the vCore/ Static IP is correctly allocated on the target business group for CloudHub deployments.

The _Runtime Manager - CloudHub Asset Name_ is the domain name of the application deployed to CloudHub as below. The application must be in "Started" state for it to be migrated

![image](/images/img-am8.png)

Once migrated the application is deployed with a "-m" suffix since we didn't delete the same application in the source business group and environment.

![image](/images/img-am9.png)

The source application is stopped and remains in the Undeployed state

![image](/images/img-am10.png)

Now this migration would mean that the application name has changed and the clients would have to change their end point URLs to access this endpoint. So in order to prevent any impact to the client a new DLB Mapping rule MUST be added for the specific application so that it now routes the request to the new migrated application with the "-m" suffix

![image](/images/img-am11.png)

Once Step 2 is complete the DLB Mapping rule MUST be removed in order for requests to be routed to the correct application using the previously existing mapping rules. 

The DLB Mapping rule changes can be skipped if a 1-step migration is followed for this

> _NOTE: The API supports migrating applicatins with Static IP and will aim to maintain the same Static IP in the new business group and environment on a best efforts basis. Usually this will work in most cases. However, maintaining the same Static IP cannot be aboslutely guaranteed because there is a brief window (~5 mins) when the Static IP is released back to the pool before reassigning back to the migrated app and during this window this can be acquired by a different EC2 instance trying to reserve a Static IP in the same region by any AWS tenant (not just MuleSoft CloudHub/ Customer specific)_

##### RTF

The _Runtime Manager - RTF Asset Name_ is the name of the application deployed to RTF Cluster as below. The application must be in "Running" state for it to be migrated

![image](/images/img-am12.png)

Once migrated the application is deployed with a "-m" suffix since we didn't delete the same application in the source business group and environment.

![image](/images/img-am13.png)

The source application is stopped and remains in the Not Running state

![image](/images/img-am14.png)

Now this migration would mean that the application name has changed but the clients can continue to use the same end point URLs to access this endpoint because the "Application url" remains the same for both source and target applications.

![image](/images/img-am15.png)

Also, no further action is required once Step 2 is completed.

The 1-step migration process can also be followed for this if required.
