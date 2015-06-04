The Problem
===========

I wanted customers to be able to click a link that would *automatically apply a discount code* once the customer reaches our checkout. In practice this could be used for Marketing Emails (eg. Click this button for 15% off!), on Google Adwords ads, Facebook posts, etc. This makes things easier for the customer because they don’t have to manually enter discount codes and it also makes things easier for us because we can track *every single customer that clicks that link and places and order*.

***

Intro
=====

Although this should work well in a large variety of themes, this code has only been tested on our store (which uses the Minimal Theme), so just keep in mind the code in your theme may look slightly different. It’ fairly basic code though and only requires copying and pasting 3 small code snippets into 2 of your theme files.

I wanted write this tutorial as just a small way of paying it forward for all the great information on this forum I’ve learned from. This discussion was started [here](https://ecommerce.shopify.com/c/shopify-discussion/t/auto-fill-discount-from-url-135012) and huge props to Patrick, who helped **tremendously** in getting me started.

<small>Disclaimers: My Javascript skills are complete rubbish so this code is provided without any sort of guarantee. Yes, I realize not all browsers support the HTML5 Storage Spec, but I couldn’t be troubled with supporting unspeakable browsers like IE 6 or 7. Others will almost certainly find ways of simplifying this and any suggestions for improvement are welcome. This is an advanced'ish type of customization we are doing so I wouldn’t expect Shopify, me, or anyone else here to fix anything if you try this and it kills your pet koala and/or your computer explodes. I’m sure myself and others here will try and help when/if we can but it should definitely not be expected. Now that we have that out of the way let’s get going!</small>

***

Overview
========

Here is a basic outline of what we are doing:

1. Customer visits a link that contains our Discount Code on the end of it — eg. **www.YourStore.com/?discount=15percentoff**

2. We grab that discount code from the Query String and do 2 things with it. Note: The query string is the **?discount=15percentoff** part of the URL

	1. We save it as a HTML5 sessionStorage Variable.
	2. We add it to the contents of a hidden input box. Why do we do this? because *if the discount in the query string* and *the discount stored in our hidden input box* are different, then we need to clear the sessionStorage variable before we can set it again.

	Notes: Once a sessionStorage variable is set it will persist until the customer closes the browser window/tab. When the browser window/tab is closed all sessionStorage variables are **cleared**, so in order to apply the discount again the customer would need to click the same link again. We could have used a HTML5 *localStorage* Variable but then the customer would have basically been given the same discount code on every subsequent visit until the end of time—and that’s probably not what we want.
	
3. When the customer visits the cart page of our Shopify store we grab the contents of that sessionStorage variable and add that to the cart form’s *action* parameter. So when the customer clicks the **checkout** button we send them to a link that looks like https://checkout.shopify.com/99999/checkouts/99999999**?discount=15percentoff**. This is the step that makes the magic happen.

***

Step-by-Step
============

1. Copy and paste the following code into your *theme.liquid* file right **before** the closing `</head>` tag. This code is commented to the best of my [very limited] ability.

	```
	<script>
	/* Begin Discount Autofill Code */
	/* Read the Query String value from the URL */    
	/* Link: http://stackoverflow.com/a/2880929 - Notes: No clue how this regex stuff works. Probably fairy magic.  */
		var urlParams;
		(window.onpopstate = function () {
		    var match,
		        pl     = /\+/g,  // Regex for replacing addition symbol with a space
		        search = /([^&=]+)=?([^&]*)/g,
		        decode = function (s) { return decodeURIComponent(s.replace(pl, " ")); },
		        query  = window.location.search.substring(1);

		urlParams = {};
		while (match = search.exec(query))
		urlParams[decode(match[1])] = decode(match[2]);
		})();

	/* If the value of the hidden discount input field is blank, undefined or null then set the contents of localstorage discount to be the result of our Query String function above */    
		if ($("#discount_input").val() == ('' || undefined || null || 'undefined')) {
		/* Set sessionStorage variable "discount" to the result of our Query string function above */
		   sessionStorage.setItem("discount", urlParams["discount"]);
		}

	/* If the value in our hidden input field doesn’t match the result of our Query String function above then clear the current sessionStorage value for "discount" and replace it with the new one */
	/* Link: http://stackoverflow.com/q/19844750 */
		else if ($("#discount_input").val() != urlParams["discount"]) {
		   sessionStorage.removeItem("discount");
		   sessionStorage.setItem("discount", urlParams["discount"]);
		}    
/* End Discount Autofill Code */
	</script>
	```

2. Copy and paste the following code into your *theme.liquid* file right **after** the opening `<body>` tag.

	```
	<!-- Our hidden input that stores the discount code value -->
	<input id="discount_input" type="hidden" name="discount" value="">
	<!-- Set the value of our hidden input field #discount-input to the value of our sessionStorage "discount" Variable -->
	<script>document.getElementById("discount_input").value = sessionStorage.getItem("discount");</script>
	```

3. Copy and past the following code into your *cart.liquid* file right **after** the `<form action="/cart" method="post" id="cartform">` tag.

	```
	<script>
   /* If the length of sessionStorage "discount" is NOT 0 then add our sessionStorage value to the end of our Checkout link */
   /* Link: http://stackoverflow.com/a/4704786 */
	if (sessionStorage.getItem("discount").length != 0) {
      document.getElementById('cartform').action = "/cart?discount=" + sessionStorage.getItem("discount");
   };
   </script>
	```

Done!
=====
Go to www.*YourStoreHere*.com/?discount=*YourDiscountCodeHere* Add something to your cart, go to your cart page, and  then click the **checkout** button. If everything went well you should see your discount code automatically applied to your order.

Did this work on your store or do you have suggestions for improving it? Leave a comment below! Thanks!