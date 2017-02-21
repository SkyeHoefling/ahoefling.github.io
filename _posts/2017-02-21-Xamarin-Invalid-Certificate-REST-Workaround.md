---
layout: post
title:  Xamarin.Forms Untrusted Certificate SSL Development Workaround
modified: 2017-02-21 13:00:00
categories: Xamarin
tags: [Development, Invalid Certificate, SSL, HTTPS, Untrusted, C#, .NET]
share: true
comments: true
---
On my Xamarin.Forms project I am using an Untrusted Certificate (SSL) just for development on my local machine and with the development servers. This has caused several headaches and issues while trying to code around it on VPN. The latest issue that has popped up was exceptions being thrown regarding untrusted certificates when trying to access images off of the development server.

{% highlight c# linenos %}
[0:] Image Loading: Error getting stream for http://dev.enviornment.com/sample/image.jpg: 
System.Net.WebException: Error: TrustFailure (The authentication or decryption has failed.) ---> 
System.IO.IOException: The authentication or decryption has failed. ---> 
System.IO.IOException: The authentication or decryption has failed. ---> 
Mono.Security.Protocol.Tls.TlsException: Invalid certificate received from server. Error code: 0xffffffff800b010b
{% endhighlight %}

This exception can pop-up in multiple forms but the general idea remains the same:

 - Invalid Certificate
 - Untrusted Certificate
 - Trust Failure

## Simple Fix ##
There is a simple solution to this and you can work around the problem to continue moving forward with your development with the Invalid Certificate. In your device specific projects you will need to add Certificate Validation code. In our example below we added code to the MainActivity in the Droid project.

New Line:
{% highlight c# linenos %}
ServicePointManager.SErverCertificateValidationCallback += (o, cert, chain, errors) => true;
{% endhighlight %}

Entire Snippet:
{% highlight c# linenos %}
public class MainActivity : FormsApplicationActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);
        Forms.Init(this, bundle);

        // I strongly recommend wrapping this in the compiler directive, because you should have a proper
        // certificate in a production environment.
#if DEBUG
        ServicePointManager.ServerCertificateValidationCallback += (o, certificate, chain, errors) => true;
#endif

        LoadApplication(new MkoFormsApplication());
    }
}
{% endhighlight %}

This workaround is just to unblock development and is not intended to go into production hence the compile directives `#if DEBUG`.
