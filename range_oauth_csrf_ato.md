# Account takeover because of an oauth csrf

## How:

The application I was testing provides a login through google, microsoft, slack, zoom and others, but it provides also for the user to link his account
with a zoom, slack or other applications, so what did I find?

![alt text](https://www.websequencediagrams.com/cgi-bin/cdraw?lz=CiAgICBVc2VyLT5DbGllbnQ6IEdFVCAvcGVlawpwYXJ0aWNpcGFudCBBdXRoU2VydmVyCgphY3RpdmF0ZQA0BQAECgA7BgoKAEMGLT5Vc2VyOiAzMDI6IGxvY2F0aW9uPWF1dGgvYXV0aG9yaXplCmRlAD4OAAULAEYIAIEXBgB1CgCBGgcAOgoAdxcAgSYLAIEyCgCBDQh7bWVzc2FnZXM6ICJEbyB5b3UgYXBwcm92ZT8ifQB6HACBewwAgQgSAD0HAF41AIIaDmMAgwgFL2hhbmRsZV9jb2RlP2NvZGU9ZGtzaGZqZwB1LgCDTQ0AOhkAgy8nAIJ-DFBPU1Q6IC90b2tlbgA2FwCCcBcAhGUIMjAwOiB7YWNjZXNzXwBDBTpDTk1CVkNYS1ZZfSAAgmMYAIRbCFJlc291cmNlAIQODXIADgcoAEsMKQCFLAoAJw4KADYOAIEMD3Jlc3BvbnMAhR0NADEPAIVhDwAvCHVsdACFPBMAhWEQAIZ7BQoKCg&s=qsd)

So as you can see when a client application tries to make an oauth request to the oauth provider it sends a parameter called state, this parameter is 
responsible for indicating which user established this request, so after the user giving the consent to the client application the oauth provider or 
server send the user back to his client application with the same state value in the url after this the client application check this value is the same
to the value that this user created when initializing the oauth request.

But what if the application didn't check the state value, this means that the application can't make sure that the response is definitly came from the 
same the oauth request flow, so let's remember our application says that I can link my account with a zoom or slack then I can login using my zoom or 
slack account so all I need to do is to make the user link his account to my zoom account and by this I can login to the victim account using my zoom
account.

So let's track the oauth flow:

- First I need to initialize the oauth request.
- After giving the consent to the client app the oauth server will redirect me with a request looking like this:
### https://[domain]/login/with-zoom?code=Vf7pjiOfF9_gBLVdXr_SFaqumySXEbnnA&state=eyJrZXkiOiJvYXV0aF9pVm1nLUtFLTJnUW1HZHNRQVhKVGtBIiwibW9kZSI6Imluc3RhbGwtem9vbSIsInV0bVBhcmFtcyI6e30sImdlbmVyYXRlZEJ5IjoiYXBpUmVkaXJlY3QifQ%3D%3D
- Then I need to delete the state value:
### https://[domain]/login/with-zoom?code=Vf7pjiOfF9_gBLVdXr_SFaqumySXEbnnA&state=
- After this I need copy the url and drop the request
- Then I need to deliver it to the victim and after he visits it the client application will make a post request with the oauth code:
### POST https://[domain]/v1/users/AAoBFu1zEshx3C3GAgAU/integrations/zoom
### Data: {"code":"Vf7pjiOfF9_gBLVdXr_SFaqumySXEbnnA"}
- After this the application will link the victim account with my zoom account and now I can login to the victim account using my zoom account
- And this is an account takeover.

### Range Hall Of Fame : https://www.range.co/security/hall-of-fame (as Mekky)

### My Info :

#### Intigriti : https://app.intigriti.com/researcher/profile/mekky

#### Linkedin : https://www.linkedin.com/in/muhammed-mekkawy-1504821b2/

#### Twitter : https://twitter.com/Mekky49295157
