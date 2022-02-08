# Lendica Documentation

- [Lendica Documentation](#lendica-documentation)
  - [iTab Guide](#itab-guide)
    - [Installation](#installation)
      - [Include IIFE bundle in index.html](#include-iife-bundle-in-indexhtml)
    - [Initialization](#initialization)
    - [Open iTab](#open-itab)
    - [Open iTab on invoice page](#open-itab-on-invoice-page)
    - [Invoice API](#invoice-api)
    - [Hiding iTab](#hiding-itab)
    - [Move to Production](#move-to-production)


## iTab Guide

### Installation

Getting set up is very simple. To begin, include the IIFE bundle directly into the html page(s) you'd like to install the iTab.

#### Include IIFE bundle in index.html

```html
<script src="https://static.golendica.com/itab.js"></script>
```

### Initialization

Renders iTab to the screen.

```javascript
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
### Open iTab

```javascript
lendica.open();
```

### Open iTab on invoice page

```javascript
lendica.open(invoiceId);  // invoiceId: unique identifier of the invoice in partner system
```

### Invoice API

> #### Get invoice

```javascript
lendica.invoices.getById(invoiceId); // invoiceId: unique identifier of the invoice in partner system
```

Returns invoice data OR Promise that will resolve with invoice data once widget is loaded.
Returns undefined if no invoice with such id is found.
Returns `{ activated: false }` if Lendica is not activated.

Invoice data schema:
```javascript
{
    order_id: "12345", // unique id of the invoice passed from partner system
    order_timestamp: "yyyy-mm-ddThh:mm:ss",
    invoice_id: "uuid", // Lendica's internal unique invoice reference id
    total_price: 123.45,
    status: 0, // 0 - Not paid, 2 - Pending delivery, 3 - Delivered, 4 - Completed
    action: 0, // 0 - Get paid now, 2 - Confirm delivery, 3 - See progress, 4 - View details
}
```

> #### Subscribe to invoices load event

```javascript
lendica.invoices.onLoad(() => {});
```

Passed callback is going to be called when invoices finished loading. Not called for invoice updates.
Returns callback to dispose subscription.

> #### Subscribe to invoices change event

```javascript
lendica.invoices.onChange(() => {});
```

Passed callback is going to be called when there was any change to invoices during page lifecycle. Subscription is not called for initial load of invoices.
Returns callback to dispose subscription.

### Hiding iTab

To allow a customer to hide the iTab from your web application you may call the destroy method. This will remove all DOM nodes associated with the iTab and clear memory.

```javascript
lendica.destroy();
```

> #### Showing iTab after hidden

To allow the customer to unhide the iTab, you may simply call the init function again with the parameters discussed above.

Call `lendica.init` with credentials and config again to re-initialize the iTab

### Move to Production

Once the testing is done and you are ready for production, please specify "PRODUCTION" in the "environment" variable. The iTab script will automatically call routes in production environment.