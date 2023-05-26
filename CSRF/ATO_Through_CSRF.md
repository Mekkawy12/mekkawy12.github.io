# Account Take Over Through A CSRF Combined With An Invitation Feature & The Login Functionality.

## How

While testing the api I thought about downloading the apk version of this app to see if there are another endpoints that I can test it further. After decompiling the apk I
noticed an endpoint that changes the email of the user and I noticed that this endpoint is open to <span style="color:red">***CSRF***</span> vulnerability, but The 
endpoint needed an additional information for the attack to be successful and I was able to get it using the invitation feature. After this I used the login functionality to deliver the attack with a better way and eleminate the factor of complexity.

# The CSRF Vulnerability

- After decompiling the apk I have an endpoint like this <span style="color:red">***/v2/account***</span>.
- This endpoint expected to be sent as a <span style="color:red">***POST***</span> request with a this body:
  <span style="color:red">***action=requestemailupdate&email=&accountid=***</span><br>
- So I tried to send this request as a <span color="red">***GET***</span> request and it was successful. All I needed is to provide the accountid of the user doing this action and a new email.
- So I wanted to first identify if I can craft a simple form with a <span style="color:red">***GET***</span> request to validate the CSRF vulnerability and it was successful.
- This attack worked because The cookie was protected with just the <span style="color:red">***LAX***</span> samesite flag and I was able to send it as a <span style="color:red">***GET***</span>
  request and the application didn't really check on anything else.
- But as you can see if want to target a specific user how I need to know his accountid.
- So how I am going to get his accountid?

  ![alt text](email_update_flow.png)

# The Invitation Feature

- The application had an invitation feature to invite other user to compete on different things.
- I was able to know this from the apk also because I was able to find an invitation endpoint <span color="red">***/v2/invitation***</span>
- This endpoint expected to be sent as a <span color="red">POST</span> request with this body:
  <span style="color:red">action=getallrecieved</span>.
- From the name of the action I anticipated that it gets the invitations sent to me.
- So I thought can I send invitations? lets try another action what about <span style="color:red">***send***</span> but the application responded with <span style="color:red">***not implemented***</span>.
- What about <span style="color:red">***create***</span>. The application responded that it needs a parameter called <span style="color:red">***email***</span>.
- Suppling the email The request was successful and I recieved the invitation. But the response was an empty body.
- But if I can there is an action called <span style="color:red">***getallrecieved***</span> maybe there is an action called <span style="color:red">***getallsent***</span>.
- And yes with this action I recieved a response with all the invitations I sent with the email and the <span style="color:red">***accountid***</span> of the user.
- With this the <span style="color:red">***CSRF***</span> attack was ready to be delivered. But can I find a better way to deliever this attack?

  ![alt text](create_invitation_flow.png)

  ![alt text](get_all_invitations_flow.png)

# The Login Functionality

- The application gives you the option to login by providing your email then recieving a magic link to click and sign you in.
- This endpoint expects to recieve a parameter called <span style="color:red">***r***</span> to be sent as a body or as query.
- This parameter is a redirection parameter that redirects the user to an endpoint he wants after logging.
- The application appends the parameter to the magic link so as soon as you click it the application sign you in then redirects to the value of <span style="color:red">***r***</span>.
- So it's a feature really no open redirect here but we can use it to make the user be redirected to the csrf endpoint and change his email to our email.
- All I have to do is to put the <span style="color:red">***CSRF***</span> payload as value to the <span style="color:red">***r***</span> parameter like this:
  <span style="color:red">***r=/v2/account?action=requestemailupdate&email=[OUR_EMAIL]&accountid=[THE_VICTIM_ACCOUNTID]***</span>.
- Sending this with the login request the applicaiton is going to append it to the magic link and as soon as the user click it the application is going to sign him in
  then is going to redirect him to the <span style="color:red">***CSRF***</span> endpoint to change his email and with the we were able to takeover the user account.
  
  ![alt text](magic_link_flow.png)
  
## My Info
  
#### Yeswehack  : <a href="https://yeswehack.com/hunters/mekky">https://yeswehack.com/user/mekky</a>

#### Intigriti  : <a href="https://app.intigriti.com/researcher/profile/mekky">https://app.intigriti.com/researcher/profile/mekky</a>

#### Linkedin   : <a href="https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/">https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/</a>

#### Twitter    : <a href="https://twitter.com/Mekky49295157">https://twitter.com/Mekky49295157</a>

