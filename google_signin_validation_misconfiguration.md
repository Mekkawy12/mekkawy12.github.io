# Google Implicit Grant Validation Misconfiguration

## How:

Some applications uses google oauth implicit grant type to signin users to it's application. By this google generates JWT and the developer has to validate it in some way but 
he didn't do this with the proper way. So I was able to use this misconfiguration to takeover the user account. The attack is complex this is why it's medium but it worth for
me to keep it and try it as an attack vector.

## Why I thought about it?

After spending sometime with oauth code authorization grant type to see what attack vectors can be there in order to find a vulnerability. I saw other applications denpend on 
google oauth implicit grant type in order to signin users. So I was curious really what attack vectors can be there?.

## So what is JWT?

Json web token is a new way of sharing information between two entities. It offers a way to make sure that you are the issuer of this token. Applications use this token to 
identify your identity.

### So what JWT consists of?

- Header: {"alg":"HS256"}
  The header identifies some of the JWT information like what algorithm is used in order to sign this JWT. What public key I can use in order to verfiy this token and others.
  
- Payload: {"email":"g@gmail.com"}
  The paylaod has some of the requested information. Or the information that the applications validate in order to know your identity.
  
- Signature:
  The signature is the one that tells me that this token belongs to me as the one how issued this token. It also confirms for the other third party that may request this token
  that it's generated by me.
  
### So how JWT is formed?

