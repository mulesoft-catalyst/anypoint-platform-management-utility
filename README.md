# Anypoint Platform Management

The Anypoint Platform Management API is used to perform various automation activities on the platform and can be invoked either from Postman or from the CI/ CD pipelines by passing an access_token either that of a user / connected app

Since, the API works with an access token, the platform management activities that can be performed using this API will depend on the level of authorization the user or the connected app has. This applies to any endpoint described below and references to "all/ any business groups/ environments" in the description implies only the resouces which the token is authorized to access. Please follow the instructions to generate an access token using the [Anypoint Platform - Access Management API](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-23abe036327a.anypoint-platform/access-management-api/minor/1.0/pages/Authentication/)

The functionality of this API has been implemented as a Mule application and can be deployed on any runtime platform CloudHub, RTF or Standalone and can be accessed in the same way as any other apps on the platform. This source code is also available for any customisation/ bug fixes as required. The app can also be run on Anypoint Studio to validate the functionality of this utility. The following instructions were created by running the application locally on port 8081.

**Base URL:** http://localhost:8081/api/

**Security:** Obtain an Access Token from the Anypoint Platform and pass it as a Bearer token in the HTTP Authorization header

![image](/images/img1.png)

## Management Plane

All operations for the Management Plane can be seen as sub resources under the \_/management-plane \_endpoint

## Get Info

The Get Info endpoint returns information on the Business Groups and Environments defined under the Anypoint Platform Master Organization. The information that is currently returned are:

- Business Group Name
- Business Group Id
- Number of allocated Production vCores
- Number of allocated Sandbox vCores
- Number of allocated Design vCores
- Number of allocated Static IPs
- Environment Names
- Environment Id

This endpoint can be invoked by caling

```
GET /management-plane/info
```

By default the API will return all business groups and all environments present within the Anypoint Platform Master Organisation in JSON format. However, the data can be filtered by passing an array of Business Group and/ or Environment names as a query parameter. Also, it can return the data in a CSV format by passing an "Accept" http header in the request

For eg:

```
curl --location -g --request GET 'http://localhost:8081/api/management-plane/info?businessGroups=[Operations, C4E]&environments=[Development, Test]' \
--header 'Accept: application/csv' \
--header 'Authorization: Bearer xxxxxxxxxxxxxxxxxx'
```

### Users

TBC

### Asset Migration

This information is available in the [Asset Migration](ASSET-MIGRATION.md) page

### Asset Copy & Purge

This information is available in the [Asset Copy & Purge](ASSET-COPY-AND-PURGE.md) page

## Runtime Plane

### Get CloudHub Applications

The Get CloudHub Applications endpoint returns information on all the Applications defined under any business group or any environments within the Anypoint Platform Master Organization. The information that is currently returned are:

- Name
- Business Group Name
- Environment Name
- Status
- Runtime Version
- Number of Workers
- CPU
- Memory

This endpoint can be invoked by caling

```
GET /runtime-plane/CloudHub/applications
```

By default the API will return applications from all business groups and environments present within the Anypoint Platform Master Organisation in JSON format. However, the data can be filtered by passing an array of Business Group and/ or Environment names and/or Application Name as a query parameter. Also, it can return the data in a CSV format by passing an "Accept" http header in the request

For eg:

```
curl --location -g --request GET 'http://localhost:8081/api/runtime-plane/CloudHub/applications?businessGroups=[C4E]&environments=[Development]&applications=[exp-example-api]' \
--header 'Accept: application/csv' \
--header 'Authorization: Bearer xxxxxxxxxxxxxxxxxxx'
```

### Get RTF Applications

The Get RTF Applications endpoint returns information on all the Applications defined under any business group or any environments within the Anypoint Platform Master Organization. The information that is currently returned are:

- Name
- Business Group Name
- Environment Name
- Status
- Runtime Version
- Number of Replicas
- CPU Reserved
- Memory

This endpoint can be invoked by caling

```
GET /runtime-plane/RTF/applications
```

By default the API will return applications from all business groups and environments present within the Anypoint Platform Master Organisation in JSON format. However, the data can be filtered by passing an array of Business Group and/ or Environment names and/or Application Name as a query parameter. Also, it can return the data in a CSV format by passing an "Accept" http header in the request

For eg:

```
curl --location -g --request GET 'http://localhost:8081/api/runtime-plane/RTF/applications?businessGroups=[C4E]&environments=[Development]&applications=[sys-example-api]' \
--header 'Accept: application/csv' \
--header 'Authorization: Bearer xxxxxxxxxxxxxxxxxxx'
```

### Patch RTF Applications

The Patch RTF Applications endpoint allows to patch the runtime version on all the Applications defined under the Anypoint Platform Master Organization. The endpoint needs to be invoked with the following information:

- Runtime Version (Array of all runtime versions to be patched)
    - From - The runtime version (eg: 4.2.2)
    - To - The runtime version (eg: 4.2.2:20210915-5)

This endpoint can be invoked by caling

```
PATCH /runtime-plane/RTF/applications
```
> **IMPORTANT:**` Always use fitering when applying any patch on RTF as this will trigger the patching on all replicas at the same time and may causing issues with resource contention`

By default the API will patch applications from all business groups and environments present within the Anypoint Platform Master Organisation. However, the request can be filtered by passing an array of Business Group and/ or Environment names and/or Application Name as a query parameter to limit the number of applications to be patched in one request.
For eg:

```
curl --location -g --request PATCH 'http://localhost:8081/api/runtime-plane/RTF/applications?businessGroups=[MuleSoft]&environments=[Development]&applications=[]'\
--header 'Authorization: Bearer xxxxxxxxxxxxxxxxxxx ' \
--header 'Content-Type: application/json' \
--data-raw '{
    "runtimeVersion": [
        {
            "from": "4.2.1",
            "to": "4.2.1:20210222-15"
        },
        {
            "from": "4.2.2",
            "to": "4.2.2:20210915-5"
        },
        {
            "from": "4.3.0",
            "to": "4.3.0:20211104-2"
        },
        {
            "from": "4.4.0",
            "to": "4.4.0:20211104-4"
        }
    ]
}'
```

> Although the above endpoint can be used to upgrade the Mule runtime minor and/or patch versions, it is recommended to NOT use this process to apply an upgrade because it need to update the application POM file with the upgraded runtime version and follow a test cycle to ensure that the application is regression tested before they are deployed to the RTF cluster`
