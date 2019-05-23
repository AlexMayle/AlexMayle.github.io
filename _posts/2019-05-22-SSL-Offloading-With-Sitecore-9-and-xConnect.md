---
layout: post
title: "SSL Offloading with Sitecore 9 and xConnect"
tags: [Sitecore, "Sitecore 9", "SSL", "AWS", "SSL Offloading", "Load Balancer"]
description: "How to disable the client certificate requirement for xConnect and implement SSL offloading. Maintain security, but in a smarter, more managable way."
---
I'll open with a discussion about whether or not you should do this, and why I'm not crazy. If you're already set on implementing SSL offloading, than skip to the second section. 

# Obligatory Justification for Breaking the Rules

So the first thing you are probably thinking, if you are from the Sitecore world, is, "why would you disable client certificates?" Indeed, it is a strong recommendation from Sitecore. They do not tell you how to disable the client-side certificate requirement of Sitecore, and they note, numerous times, that it is mandatory for production environments. Much like any recommendation, there are exceptions, and depending on your organization, this may, or may not, be the best way to manage secure communications between xConnect and its clients.

No one is suggesting that we should forgoe HTTPS, but rather, implement it in a more managable way - if your situation allows it. Let's look at the one major requirement your organization will need to meet if you are to find any value in this article. 

 - No third-party clients will access xConnect.

Now, obviously, xConnect was created with the goal of facilitating third-party ingestion of users' analytical data. Absolutely requiring client-side certificates out-of-the-box definitely sets you up for flexibility, but there is also something (negative) to be said about maintaining features you don't use. Note to Sitecore: add a config element for this. Not everyone is dishing off their data analysis to vendors.

Even if your organization meets the above requirement, there should be some concrete reasons for straying away from the Sitecore recommendation. This is highly tailored to AWS, but may apply to other platforms. 

1. AWS doesn't let you deploy Amazon Certificate Manager (ACM) certs to individual instances. So have fun managing those at the OS level. Say goodbye to automatic renewals and say hello to traditional certificate issuer fees.
2. While it is possible to implement SSL pass-through on AWS load balancers, it is more laborious than SSL offloading
3. Servers will take on the load of TLS/SSL handshakes. `(Marginal) ? "not a bad idea" : "good idea"`

It comes down to flexibility vs complexity. If the flexibility of having arbitrary clients is worth the complexity of manually renewing certificates, setting up TLS pass through at the load balancer level, and eating some CPU & I/O resources, than go for it. If it's not, continue reading.

# Pitter Patter

This is actually pretty simple. The first part creates an SSL offloading load balancer. This will setup a secure tunnel between the client and the load balancer. It will establish a TLS connection between the client and load balancer. The request will be forwarded to the target instance over plain old HTTP. The response will be forwarded back to the client over the same TLS channel. Point being, the load balancer and client communicate over HTTPS, but the target instance and the load balancer communicate over HTTP. This is SSL offloading. If you don't trust the connection between the load balancer and target, than you don't trust your platform, and should switch. (Or you are compelled to comply with HIPPA and other strict regulations, which probably don't apply to you. If they do, don't do this.)

## Load Balancer setup

This is for AWS, but if you're on another platform, the general gist is the same. 

1. Setup a load balancer which listens on port 443. 
2. Install the server side certificate. If you're on AWS just deploy an ACM cert.
3. Setup a target group for port 80 and place an xConnect server in it.

## Disable HTTPS enforcement and client-certificate inspection

We now need to disable the inspection of the client certificate, as well as the enforcement of HTTPS. The latter may seem extreme, but this is the beauty of SSL offloading. Let the load balancer do the work and allow the server itself to simply avoid the TLS handshakes. Both of the following files can be found in the `<site-root>\App_data\Config\sitecore\CoreServices` directory.

Disable `sc.XConnect.Security.EnforceSSL.xml` by commenting out the file, as seen below. One may be able to just add the `.disabled` suffix, which would probably be preferrable - as it exposes the fact that it's disabled from the file system itself - but I did not test this. 

```xml
<Settings>
  <!--
    Provides default configuration for component services for the application    
        
    NOTE:
      When registering a type you can configure the following values:
        Type      (Required): The full type reference of the instance being registered
        As        (Optional): The type the instance may be resolved as (defaults to Type if not provided)
        LifeTime  (Optional): The lifetime of instance.  Can be Singleton or Transient. (defaults to Singleton)
        Options   (Optional): If the type contains a constuctor that recieves IConfiguration, you can proivide
                              addtional values here to match the resulting type of options for that type.
  -->
  <Sitecore>
    <XConnect>
      <Services>
<!--         <SslValidationHttpConfiguration>
          <Type>Sitecore.XConnect.Security.Web.SslValidationHttpConfiguration, Sitecore.XConnect.Security.Web</Type>
          <As>Sitecore.XConnect.DependencyInjection.Web.Abstractions.IHttpConfiguration, Sitecore.XConnect.DependencyInjection.Web</As>
          <LifeTime>Transient</LifeTime>
        </SslValidationHttpConfiguration> -->
      </Services>
    </XConnect>
  </Sitecore>
</Settings>
```

Disable `sc.XConnect.Security.EnforceSSLWithCertificateValidation.xml` in the same fashion. 

```xml
<Settings>
  <!--
    Provides default configuration for component services for the application    
        
    NOTE:
      When registering a type you can configure the following values:
        Type      (Required): The full type reference of the instance being registered
        As        (Optional): The type the instance may be resolved as (defaults to Type if not provided)
        LifeTime  (Optional): The lifetime of instance.  Can be Singleton or Transient. (defaults to Singleton)
        Options   (Optional): If the type contains a constuctor that recieves IConfiguration, you can proivide
                              addtional values here to match the resulting type of options for that type.
  -->
  <Sitecore>
    <XConnect>
      <Services>
<!--         <CertificateValidationHttpConfiguration>
          <Type>Sitecore.XConnect.Security.Web.CertificateValidationHttpConfiguration, Sitecore.XConnect.Security.Web</Type>
          <As>Sitecore.XConnect.DependencyInjection.Web.Abstractions.IHttpConfiguration, Sitecore.XConnect.DependencyInjection.Web</As>
          <LifeTime>Transient</LifeTime>
        </CertificateValidationHttpConfiguration> -->
      </Services>
    </XConnect>
  </Sitecore>
</Settings>
```

# Restricting Client Access Without Client Certificates

Simply use IP restrictions. If you control the clients, than you know the IPs. Better yet, keep your clients in particular subnets and simply add a CIDR block to your firewall. That way even a dynamic pool of servers can access the xConnect services. If you're on AWS, use security group rules which reference other security group rules to achieve this. Add redundancy by spficiying the CIDR block rules at the ACL level.

# Conclusion

While this post is heavily tilted toward AWS, which is not the preferred platform for Sitecore, the guidlines still apply. Your platform probably enables you to manage certificates in a smarter way than installing them at the OS level. Your platform probably provides facilitaties for offloading the load of TLS handshakes to a router, load balancer, or similar. Use these if you don't need to manage clients which you have no control over. 
