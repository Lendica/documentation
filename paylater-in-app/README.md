# Lendica iBranch Application and PayLater-in-App

- [Frontend Integration](#frontend-integration)
  - [iBranch and PayLater API](#use-ibranch-and-paylater-api)
  - [Lendica PayLater button](#use-the-lendica-paylater-button)
- [Backend Integration](#backend-integration)
  - [Buyer PayLater application link](#buyer-paylater-application-link)
  - [Approval summary callback route](#approval-summary-callback-route)

# Frontend Integration


## Use iBranch and PayLater API


### Installing iBranch

Include IIFE bundle in index.html

```jsx
<script src="https://static.golendica.com/iBranch.js"></script>
```

### Initialize Lendica

Renders iBranch to the screen.

```jsx
lendica.init({
    // Required partner and user credentials
    partner_name: 'Partner', // Technology vendor name
    company_id: 'COMPANY_ID', // Unique client id in technology vendor's system
    company_name: 'COMPANY_NAME', // Client's business name
    company_access_token: 'ACCESS_TOKEN' // Token/key to retrieve client data from vendor's API
}, {
    // Optional config
    access_type: 'ADMIN', // User access type, currently supporting 'VIEW_ONLY' or 'ADMIN'
    environment: 'SANDBOX' // Default environment is sandbox if not specified, currently supporting 'SANDBOX' or 'PRODUCTION'
});
```

The lendica instance will be ready in the window object once initialized.

### Open iBranch on application page

```jsx
lendica.apply();
```

### Open PayLater in app

```jsx
lendica.paylater.openInApp(url, onComplete);
```

Parameters

1. url - pass the [buyer PayLater application link](#buyer-paylater-application-link)) 
2. onComplete - function to be called when in-app popup is closed. Use the [approval data callback route](#approval-summary-callback-route) to retrieve company approval data

## Using the Lendica PayLater button

[![npm version](https://img.shields.io/badge/npm-v1.0.0-8c8ca1)](https://www.npmjs.com/package/@lendica/paylaterbutton) <img src="https://lendica-public.s3.amazonaws.com/assets/paylater_btn_light.png" height=36>

```sh
npm i @lendica/paylaterbutton
```
```javascript
import '@lendica/paylaterbutton';
```
```html
<paylater-button height="32" onclick="clickHandler()"></paylater-button>
```

See complete documentation [here](https://github.com/Lendica/paylater-button)



# Backend Integration


## Buyer PayLater Application link


```bash
{BACKEND_URL}/api/v1/pod/paylater/partner_paylater_redirect/?partner={PARTNER_NAME}&invoice_id={INVOICE_UUID}
```


- BACKEND_URL
    - This is the backend Lendica environment
    - Options:
        - https://sandbox.golendica.com
        - https://prod.golendica.com
- PARTNER_NAME
    - Options:
        - mainstem
        - mainstem_uat
- INVOICE_UUID
    - This is the internal partner UUID of the invoice for which PayLater is being offered
        - Lendica will use this ID to pull invoice details as well as seller/buyer details for the invoice from the partner API provided to Lendica
    

## Approval Summary Callback Route

### Endpoint


### POST `{{backend_url}}/api/v1/pod/approval_status`

**Request body:**

- {’`partner_company_uuid`’: `str`}
    - Pass in Mainstem’s unique identifier for the company


`backend_url` is either:

- `https://sandbox.golendica.com`
    - For demo environment
- `https://prod.golendica.com`
    - For production environment
    

## Responses


### Case 1 - Inactive

A company has not yet been initialized into the lendica system. The response will only return the `partner_company_uuid` provided in the request, and an approval status of `inactive`.

```json
{
    "error": [],
    "company": {
        "id": "",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": ""
    },
    "approval": {
        "id": "",
        "status": "inactive",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```

### Case 2 - Pending

A company has submitted a PayLater application. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `pending`.

```json
{
    "error": [],
    "company": {
        "id": "6254ca0a-e32f-11ec-90ba-0242c0a8d008",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "id": "",
        "status": "pending",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```

### Case 3 - Approved, pending approval of deal

A company has submitted a PayLater application and has been approved for a PayLater line. The company has also confirmed terms for an invoice, and a deal has been initialized in the Lendica system. Lendica will quickly process the deal, but until it is approved, the status of the deal will be `pending`. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `approved`. Now, approval details also include the live credit balance and details for the customer’s PayLater line.

```json
{
    "error": [],
    "company": {
        "id": "ba27942e-e5af-11ec-bccb-0242ac130007",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "status": "approved",
        "credit_limit": 500000,
        "credit_balance": 1960,
        "credit_available": 498040,
        "number_of_active_deals": 1
    },
    "deals": [
        {
            "id": "2e6a81ed-72d9-468a-ae96-90cf919fadf5",
            "partner_invoice_uuid": "5ec6c168-385f-4908-9571-2ba496ce3e75",
            "status": "pending",
            "product_name": "paylater"
        }
    ]
}
```

### Case 4 - Approved, Deal approved

A company has submitted a PayLater application and has been approved for a PayLater line. The status of the deal is now `approved`. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `approved`. Approval details also include the live credit balance and details for the customer’s PayLater line. 

Note that deals is a list, and will include all active PayLater deals activated by the company.

```json
{
    "error": [],
    "company": {
        "id": "ba27942e-e5af-11ec-bccb-0242ac130007",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "status": "approved",
        "credit_limit": 500000,
        "credit_balance": 1960,
        "credit_available": 498040,
        "number_of_active_deals": 1
    },
    "deals": [
        {
            "id": "2e6a81ed-72d9-468a-ae96-90cf919fadf5",
            "partner_invoice_uuid": "5ec6c168-385f-4908-9571-2ba496ce3e75",
            "status": "approved",
            "product_name": "paylater"
        }
    ]
}
```

### Case 5 - Rejected

A company has submitted an application, but has been rejected. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `rejected`.

```json
{
    "error": [],
    "company": {
        "id": "6254ca0a-e32f-11ec-90ba-0242c0a8d008",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "id": "",
        "status": "rejected",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```

## Approval Summary Callback Route


## Endpoint


### POST `{{backend_url}}/api/v1/pod/approval_status`

**Request body:**

- {’`partner_company_uuid`’: `str`}
    - Pass in Mainstem’s unique identifier for the company


`backend_url` is either:

- `https://sandbox.golendica.com`
    - For demo environment
- `https://prod.golendica.com`
    - For production environment
    

## Responses


## Case 1 - Inactive

A company has not yet been initialized into the lendica system. The response will only return the `partner_company_uuid` provided in the request, and an approval status of `inactive`.

```json
{
    "error": [],
    "company": {
        "id": "",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": ""
    },
    "approval": {
        "id": "",
        "status": "inactive",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```

## Case 2 - Pending

A company has submitted a PayLater application. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `pending`.

```json
{
    "error": [],
    "company": {
        "id": "6254ca0a-e32f-11ec-90ba-0242c0a8d008",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "id": "",
        "status": "pending",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```

## Case 3 - Approved, pending approval of deal

A company has submitted a PayLater application and has been approved for a PayLater line. The company has also confirmed terms for an invoice, and a deal has been initialized in the Lendica system. Lendica will quickly process the deal, but until it is approved, the status of the deal will be `pending`. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `approved`. Now, approval details also include the live credit balance and details for the customer’s PayLater line.

```json
{
    "error": [],
    "company": {
        "id": "ba27942e-e5af-11ec-bccb-0242ac130007",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "status": "approved",
        "credit_limit": 500000,
        "credit_balance": 1960,
        "credit_available": 498040,
        "number_of_active_deals": 1
    },
    "deals": [
        {
            "id": "2e6a81ed-72d9-468a-ae96-90cf919fadf5",
            "partner_invoice_uuid": "5ec6c168-385f-4908-9571-2ba496ce3e75",
            "status": "pending",
            "product_name": "paylater"
        }
    ]
}
```

## Case 4 - Approved, Deal approved

A company has submitted a PayLater application and has been approved for a PayLater line. The status of the deal is now `approved`. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `approved`. Approval details also include the live credit balance and details for the customer’s PayLater line. 

Note that deals is a list, and will include all active PayLater deals activated by the company.

```json
{
    "error": [],
    "company": {
        "id": "ba27942e-e5af-11ec-bccb-0242ac130007",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "status": "approved",
        "credit_limit": 500000,
        "credit_balance": 1960,
        "credit_available": 498040,
        "number_of_active_deals": 1
    },
    "deals": [
        {
            "id": "2e6a81ed-72d9-468a-ae96-90cf919fadf5",
            "partner_invoice_uuid": "5ec6c168-385f-4908-9571-2ba496ce3e75",
            "status": "approved",
            "product_name": "paylater"
        }
    ]
}
```

## Case 5 - Rejected

A company has submitted an application, but has been rejected. The response will return the company name, Lendica company ID, and partner company ID, and an approval status of `rejected`.

```json
{
    "error": [],
    "company": {
        "id": "6254ca0a-e32f-11ec-90ba-0242c0a8d008",
        "partner_company_uuid": "006c545e-a4f6-4e86-88ea-25319031d53d",
        "company_name": "Drift"
    },
    "approval": {
        "id": "",
        "status": "rejected",
        "credit_limit": 0,
        "credit_balance": 0,
        "credit_available": 0,
        "number_of_active_deals": 0
    },
    "deals": []
}
```