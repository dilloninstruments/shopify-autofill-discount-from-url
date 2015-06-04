<h1 id="toc_0">The Problem</h1>

<p>I wanted customers to be able to click a link that would <em>automatically apply a discount code</em> once the customer reaches our checkout. In practice this could be used for Marketing Emails (eg. Click this button for 15% off!), on Google Adwords ads, Facebook posts, etc. This makes things easier for the customer because they don’t have to manually enter discount codes and it also makes things easier for us because we can track <em>every single customer that clicks that link and places and order</em>.</p>

<hr>

<h1 id="toc_1">Intro</h1>

<p>Although this should work well in a large variety of themes, this code has only been tested on our store (which uses the Minimal Theme), so just keep in mind the code in your theme may look slightly different. It’ fairly basic code though and only requires copying and pasting 3 small code snippets into 2 of your theme files.</p>

<p>I wanted write this tutorial as just a small way of paying it forward for all the great information on this forum I’ve learned from. This discussion was started <a href="https://ecommerce.shopify.com/c/shopify-discussion/t/auto-fill-discount-from-url-135012">here</a> and huge props to Patrick, who helped <strong>tremendously</strong> in getting me started.</p>

<p><small>Disclaimers: My Javascript skills are complete rubbish so this code is provided without any sort of guarantee. Yes, I realize not all browsers support the HTML5 Storage Spec, but I couldn’t be troubled with supporting unspeakable browsers like IE 6 or 7. Others will almost certainly find ways of simplifying this and any suggestions for improvement are welcome. This is an advanced&#39;ish type of customization we are doing so I wouldn’t expect Shopify, me, or anyone else here to fix anything if you try this and it kills your pet koala and/or your computer explodes. I’m sure myself and others here will try and help when/if we can but it should definitely not be expected. Now that we have that out of the way let’s get going!</small></p>

<hr>

<h1 id="toc_2">Overview</h1>

<p>Here is a basic outline of what we are doing:</p>

<ol>
<li><p>Customer visits a link that contains our Discount Code on the end of it — eg. <strong>www.YourStore.com/?discount=15percentoff</strong></p></li>
<li><p>We grab that discount code from the Query String and do 2 things with it. Note: The query string is the <strong>?discount=15percentoff</strong> part of the URL</p>

<ol>
<li>We save it as a HTML5 sessionStorage Variable.</li>
<li>We add it to the contents of a hidden input box. Why do we do this? because <em>if the discount in the query string</em> and <em>the discount stored in our hidden input box</em> are different, then we need to clear the sessionStorage variable before we can set it again.</li>
</ol>

<p>Notes: Once a sessionStorage variable is set it will persist until the customer closes the browser window/tab. When the browser window/tab is closed all sessionStorage variables are <strong>cleared</strong>, so in order to apply the discount again the customer would need to click the same link again. We could have used a HTML5 <em>localStorage</em> Variable but then the customer would have basically been given the same discount code on every subsequent visit until the end of time—and that’s probably not what we want.</p></li>
<li><p>When the customer visits the cart page of our Shopify store we grab the contents of that sessionStorage variable and add that to the cart form’s <em>action</em> parameter. So when the customer clicks the <strong>checkout</strong> button we send them to a link that looks like https://checkout.shopify.com/99999/checkouts/99999999<strong>?discount=15percentoff</strong>. This is the step that makes the magic happen.</p></li>
</ol>

<hr>

<h1 id="toc_3">Step-by-Step</h1>

<ol>
<li><p>Copy and paste the following code into your <em>theme.liquid</em> file right <strong>before</strong> the closing <code>&lt;/head&gt;</code> tag. This code is commented to the best of my [very limited] ability.</p>

<pre><code>&lt;script&gt;
/* Begin Discount Autofill Code */
/* Read the Query String value from the URL */    
/* Link: http://stackoverflow.com/a/2880929 - Notes: No clue how this regex stuff works. Probably fairy magic.  */
    var urlParams;
    (window.onpopstate = function () {
        var match,
            pl     = /\+/g,  // Regex for replacing addition symbol with a space
            search = /([^&amp;=]+)=?([^&amp;]*)/g,
            decode = function (s) { return decodeURIComponent(s.replace(pl, &quot; &quot;)); },
            query  = window.location.search.substring(1);

    urlParams = {};
    while (match = search.exec(query))
    urlParams[decode(match[1])] = decode(match[2]);
    })();

/* If the value of the hidden discount input field is blank, undefined or null then set the contents of localstorage discount to be the result of our Query String function above */    
    if ($(&quot;#discount_input&quot;).val() == (&#39;&#39; || undefined || null || &#39;undefined&#39;)) {
    /* Set sessionStorage variable &quot;discount&quot; to the result of our Query string function above */
       sessionStorage.setItem(&quot;discount&quot;, urlParams[&quot;discount&quot;]);
    }

/* If the value in our hidden input field doesn’t match the result of our Query String function above then clear the current sessionStorage value for &quot;discount&quot; and replace it with the new one */
/* Link: http://stackoverflow.com/q/19844750 */
    else if ($(&quot;#discount_input&quot;).val() != urlParams[&quot;discount&quot;]) {
       sessionStorage.removeItem(&quot;discount&quot;);
       sessionStorage.setItem(&quot;discount&quot;, urlParams[&quot;discount&quot;]);
    }    
/* End Discount Autofill Code */
&lt;/script&gt;</code></pre></li>
<li><p>Copy and paste the following code into your <em>theme.liquid</em> file right <strong>after</strong> the opening <code>&lt;body&gt;</code> tag.</p>

<pre><code>&lt;!-- Our hidden input that stores the discount code value --&gt;
&lt;input id=&quot;discount_input&quot; type=&quot;hidden&quot; name=&quot;discount&quot; value=&quot;&quot;&gt;
&lt;!-- Set the value of our hidden input field #discount-input to the value of our sessionStorage &quot;discount&quot; Variable --&gt;
&lt;script&gt;document.getElementById(&quot;discount_input&quot;).value = sessionStorage.getItem(&quot;discount&quot;);&lt;/script&gt;</code></pre></li>
<li><p>Copy and past the following code into your <em>cart.liquid</em> file right <strong>after</strong> the <code>&lt;form action=&quot;/cart&quot; method=&quot;post&quot; id=&quot;cartform&quot;&gt;</code> tag.</p>

<pre><code>&lt;script&gt;
/* If the length of sessionStorage &quot;discount&quot; is NOT 0 then add our sessionStorage value to the end of our Checkout link */
/* Link: http://stackoverflow.com/a/4704786 */
if (sessionStorage.getItem(&quot;discount&quot;).length != 0) {
  document.getElementById(&#39;cartform&#39;).action = &quot;/cart?discount=&quot; + sessionStorage.getItem(&quot;discount&quot;);
};
&lt;/script&gt;</code></pre></li>
</ol>

<h1 id="toc_4">Done!</h1>

<p>Go to www.<em>YourStoreHere</em>.com/?discount=<em>YourDiscountCodeHere</em> Add something to your cart, go to your cart page, and  then click the <strong>checkout</strong> button. If everything went well you should see your discount code automatically applied to your order.</p>

<p>Did this work on your store or do you have suggestions for improving it? Leave a comment below! Thanks!</p>
