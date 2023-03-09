# Account Takeover Through Open Redirect

## How:

The program had an endpoint that was open to redirection vulnerability. By compining this vulnerability with an authentication functionality I was able to takeover the user
account.

## Open Redirection Vulnerability:

- The application had an endpoint like this ***/logout*** that takes a ***GET*** parameter called ***r***.
- The parameter ***r*** takes an endpoint to be redirected at like this ***/logout?r=/dashboard***.
- So I tried to to put a valid host to see what it does like this ***logout?r=https://program.com/***.
- The application redirected me so, I new that I can play with it to see what else I can get.
- I tried to set the host name to another domain like ***r=https://evil.com/*** and the application responded with ***invalid redirect uri***.
- Knowing this I thought of listing some payloads to try out:<br>
  ***r=https://evil.com/program.com***<br>
  ***r=https://program.com.evil.com***<br>
  ***r=https://program.com@evil.com***<br>
  ***r=https://evil.com@program.com***<br>
  
- The last one was the road to success from this list. However, it's still no redirection because the application will always redirect to itself.
- For this we have to take a step back to see what set of characters are allowed in hostname or a url and to think how the developer develoed this functionality.
  - 
  
