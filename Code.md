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