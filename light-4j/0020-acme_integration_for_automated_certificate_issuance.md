### Summary
Integrate with acme4j to get lets encrypt certificate automatically.

### Motivation
ACME(Automatic Certificate Management Environment) is a protocol to get TLS certificates automatically within your application. 
Before acme, getting a TLS certificate from a CA (Certificate Authority) was all a manual process and the process was not
standardized. So getting a TLS certificate and setting up https in the server was a pain even for experts.

Let's Encrypt is an open and automated CA that uses the ACME protocol to provide free TLS/SSL certificates to any 
compatible client.

Most of our users are using Let's Encrypt certificates but it needs to be renewed every 3 months. Currently this is done 
manually. The automated process is that you need to expose your service to the internet and the let's encrypt will challenge your service through your domain name to confirm that you control the domain. Once it is confirmed, it will issue or renew
the certificate.

### Guide-level explanation


1.Create a module named "acme" under light-4j

2.The acme module will use acme-4j (Java client for acme protocol) to implement all required functions like certificate 
issuance,renewal etc. 

3. The server module can use the acme module to setup https.

### Reference-level explanation
[There is a spring-boot implementation that might give us some ideas](https://github.com/creactiviti/spring-boot-starter-acme) 

### Drawbacks

### Rationale and Alternatives
Implementing this feature will boost developer productivity , because they don't have to go through the process of 
getting,renewing of TLS certificates or setting up https. The ideal implementation will be where the light framework
will have in-built support for https ,so that all the apps using light framework will be https protected with no or 
minimal effort from the application developer.

### Unresolved questions
1.ACME won't work for localhost , i.e. during development how the developers  can test whether https is working or not.
2.The main limit is Certificates per Registered Domain, (50 per week,renewal doesn't have a limit). A registered domain is, 
generally speaking,the part of the domain you purchased from your domain name registrar. For instance, in the name 
www.example.com, the registered domain is example.com. In new.blog.example.co.uk, the registered domain is example.co.uk.
This will become a problem only if your are using the domains provided by Paas cloud providers like heroku, where the 
registered domain for all apps will be herokuapp.com.
