# LicenseSpring-PayPal-Integration-example
Example for LicenseSpring-PayPal integration with PHP backend.

This is an integration that will issue licenses through LicenseSpring platform for every PayPal Checkout.

## How it works?
- on your web store checkout page you include LicenseSpring Javascript library and PayPal Smart Buttons Javascript SDK in this order.  
  `index.html`:
  ```html
  <script src="http://static.kraken.epa.hr/licensespring/licensespring.js"></script> 
  <!-- IMPORTANT: change cliend-id value to your seller id (replace 'sb' with your id) because funds will be send to this account -->
  <script src="https://www.paypal.com/sdk/js?client-id=sb"></script>
  ```
  
- you add LicenseSpring library and PayPal Smart Buttons to your checkout page.  
  `index.html`:
  ```js
  const licensespring = new LicenseSpring();

  paypal.Buttons({ ... }).render('body');
  ```
  
- you add listeners for Buttons: `createOrder` (you want to create order but need to do some work before it - acquire licenses) and `onApprove` (the order has been approved - display the licenses).  
    `index.html`:
  ```js
  paypal.Buttons({
      createOrder: (data, actions) => { ... },
      onApprove: (data, actions) => { ... }
  }).render('body');
  ```
  
  - in the `createOrder` section you create information about PayPal order (to help you out, LicenseSpring library offers such method).  
    `index.html`:
    ```js
    let products = [
      {
        name: "PDF Pro",
        quantity: 2,
        price: 19.99,
        code: "PP"
      },
      {
        name: "PDF 10",
        quantity: 1,
        price: 8.99,
        code: "PT"
      }
    ];
    const orderData = licensespring.generatePayPalOrder("your-order-reference-number", "USD", products);
    ```
    
    - with that information you can issue licenses through LicenseSpring for the products specified in the order.  
      `index.html`:
      ```js
      return licensespring.acquireLicenses(orderData, "php/api/licensespring/acquire-licenses.php").then(() => {
        ...
      });
      ```
      
      - to do that, you need to set a PHP script that uses LicenseSpring PHP library which will do everything for you.  
        `php/api/licensespring/acquire-licenses.php`:
        ```php
        $webhook = new LicenseSpring\Webhook("insert-your-UUID-here", "insert-your-shared-key-here");
        $webhook->acquireLicenses($orderData);
        ```
        
    - with that information you create a PayPal order.  
      `index.html`:
      ```js
      return actions.order.create(orderData);
      ```
      
  - in the `onApprove` section you can show issued licenses to user. However, licenses are not ready yet. In order for licenses to work, an order in LicenseSpring platform needs to be placed.  
    `index.html`:
    ```js
    return actions.order.capture().then(details => {
      licensespring.createOrder(details, "php/api/licensespring/create-order.php");
    });
    ```
    
    - to do that, you need to set a PHP script that uses LicenseSpring PHP library which will do everything for you.
      `php/api/licensespring/create-order.php`:
      ```php
      $webhook = new LicenseSpring\Webhook("insert-your-UUID-here", "insert-your-shared-key-here");
      $webhook->createOrder($data);
      ```
