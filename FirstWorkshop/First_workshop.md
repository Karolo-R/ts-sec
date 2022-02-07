# TeleSoftas QA Security Learning

This is the first guide how to pawn OWASP Juice Shop

## Getting started

- Create [Herokuapp](https://heroku.com/) account
- Deploy OWASP Juice Shop from [github page](https://github.com/juice-shop/juice-shop#deploy-on-heroku-free-0month-dyno)
- Download and install [OWASP ZAP](https://www.zaproxy.org/download/)
- Download Chrome and [FoxyProxy Chrome extension](https://chrome.google.com/webstore/detail/foxyproxy-standard/gcknhkkoolaabfmlnjonogaaifnjlfnp?hl=en) or Firefox and [FoxyProxy Firefox add-on](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-basic/)

## Proxy setup

In order to be able intercept requests with ZAP, we need to configure Local Proxy and add a certificate to operating system.

- ZAP > Tools > Options > Local Proxies
enter `127.0.0.1` as the address and `8080` as a port
![1](./attachments/Pasted_image_20220204172940.png)

- Navigate to ZAP > Tools > Options > Dynamic SSL Certificate
click **Save** and put the certificate in easy accessible place.
![1](./attachments/Pasted_image_20220204173121.png)

- For mac users : Open **Keychain Access** via spotlight search (cmd + spacebar)
In the **Keychains** sidebar, select **System.** In the **Category** sidebar, select **Certificates**
![1](./attachments/Pasted_image_20220204173344.png)
Then click **File > Import Items** and go find the `.cer` file you saved earlier. Double click the certificate and in the **Trust** menu that appears, change the **Secure Sockets Layer (SSL)** setting to **Always Trust**
![1](./attachments/Pasted_image_20220204173458.png)
After that close the window, and check if the certificate has blue plus sign.
![1](./attachments/Pasted_image_20220204173725.png)

- Open FoxyProxy options, and Add New Proxy
![1](./attachments/Pasted_image_20220204180707.png)
In **General** tab enter ZAP in name,
![1](./attachments/Pasted_image_20220204180810.png)
Navigate to **Proxy Details** tab enter `127.0.0.1` as the address and `8080` as a port,
![1](./attachments/Pasted_image_20220204180915.png)
Navigate to **URL Patterns** > **Add new pattern** enter `*herokuapp.com` URL pattern and save the settings.
![1](./attachments/Pasted_image_20220207152412.png)
After that, click on FoxyProxy extension and click **Use proxies based on their pre-defined patterns and priorities**
![1](./attachments/Pasted_image_20220204181054.png)

Now you should be able to navigate in your Juice Shop and see all the requests made and received in OWASP ZAP. This will be useful later, to intercept outgoing requests from our browser.
![1](./attachments/Pasted_image_20220204181307.png)

## ZAP Forced browse

Some times there are some url's that cannot be accessed via hyperlinks in website (as there are none) or cannot be found via google search, there is a specifically made tool in ZAP - Forced Browse.
ZAP allows you to try to discover directories and files using forced browsing.  
A set of files are provided which contain a large number of file and directory names.  
ZAP attempts to directly access all of the files and directories listed in the selected file directly rather than relying on finding links to them.

For this step we will import a shorter set of possible url's as the default one takes too long to execute.

Save this file as [forcedUrls.txt](./attachments/forcedUrls.txt) :

```text
cgi-bin
education
betsie
accessibility
accesskeys
go
toolbar
-
text
tv
radio
talk
ftp
whereilive
a-z
homepage
int
bb
oth
textonly
t
mobile
admin
score-board
reviews
```

- Navigate to ZAP > Tools > Options > Forced Browse
Add custom Forced Browse file
![1](./attachments/Pasted_image_20220204182745.png)
![1](./attachments/Pasted_image_20220204183910.png)
And check if new added file is selected as a default
![1](./attachments/Pasted_image_20220204183915.png)
Save the settings, and try to attack your heroku app by: **Right click** on the site > Attack > Forced Browse Site
![1](./attachments/Pasted_image_20220204184106.png)
After forced browse completion, you should see a new directory mapped to your site
![1](./attachments/Pasted_image_20220205154727.png)
I wonder, whats there? Lets check out in the browser.

## ZAP Fuzzer

In score-board we can see, that there is a challenge to receive a coupon code from the support chatbot, and we can see that there is a **Brute Force** tag attached.
![1](./attachments/Pasted_image_20220206203852.png)
Lets try to do that, open chatbot, try to chat with it, ask for coupon code
![1](./attachments/Pasted_image_20220206204156.png)
After asking for a coupon, search for a POST request in ZAP, where you were asking for it.
![1](./attachments/Pasted_image_20220206204436.png)
**Right click** on the request > Attack > Fuzz...
![1](./attachments/Pasted_image_20220206204552.png)
Then the Fuzzer window opens. As we need only brute force application, we can leave the request body same as it is, just select space between words in request message, click Add
![1](./attachments/Pasted_image_20220206204851.png)
In the Payloads window click Add, then Add Payload window appears.
Select **Type: Empty/Null** and add repetitions count (50 should be enough)
![1](./attachments/Pasted_image_20220206205132.png)
Click **Add**, then in next window **OK**, and in the last window press **Start Fuzzer**
![1](./attachments/Pasted_image_20220206205419.png)
After that, Fuzzer tab appears, select first request, open it's **Response** and then go trough requests (arrow down button) until you find your discount coupon.
![1](./attachments/Pasted_image_20220206205732.png)
![1](./attachments/Pasted_image_20220206205815.png)
