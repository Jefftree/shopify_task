
## Shopify Code Refactor

The function `order_summary` does two things:

- Calculate the pre tax and post tax price of an order
- Render the HTML code for the summary

Ideally these two should be split into separate functions: computing the total cost and rendering the HTML

However, in the event that any existing code is the function,
the interface should be kept the same to promote code reuse.

```javascript
// Render HTML code for an order summary
function order_summary(order_object) {
  var x, y; // should be more descriptive, (eg price, tax)

    if (order_object.price_level == "free") {
        x= 0; // Inconsistent spacing
    } else if (order_object.price_level == "discount") {
        x =  order_object.price - (order_object.discount_percentage * order_object.price);
    } else if (order_object.price_level == "sale") {
        x =  order_object.price - order_object.markdown;
    } else {
        x = order_object.price;
    }
```

It would probably be better to store the variables (subtotal, tax, total) in an object, and iterate over it to generate the DOM

```javascript

    var subtotal_str;
    if (order_object.price_level == "free") {
        subtotal_str = "Subtotal: This order is free";
    } else {
        subtotal_str = "Subtotal: " + "$" + x;
    }
    if (order.taxes_applicable == true) {
        y = order_object.tax; // If order.taxes_applicable != true, shouldn't the tax be 0 anyway?
    } else {
        y = 0;
    }
```

Usually tax is calculated on the final price of a product (including discounts). How was tax calculated in the original order object but the subtotal has to be calculated in this function? Ideally these calculations should be performed in the same location

```javascript

    var tax_str;
    tax_str = "Tax: $" + y;

    var total_str = "Order total: " + "$" + x + y; // str: $amount is used multiple times. Can promote into another function

  html = "<p>" + "Product:  " + order_object.product_name  + "</p>" + "<p>" + subtotal_str + "</p>" + "<p>" + tax_str + "</p>" + "<p>" + total_str + "</p>";
  document.write("<h1> Order summary </h1>" + html);
}
```

Using document.write is generally bad practice as

- Executing after page load will result in the page being overwritten
- Unable to write at a particular location. It just appends to the existing HTML
- DOM Manipulation functions would work better for frontend rendering

It would probably be better to return the HTML to another function to process or even better pass the updated order object to the View, adhering to the MVC architecture.


