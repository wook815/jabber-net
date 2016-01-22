# Introduction #

There are two use cases for connecting to GoogleTalk.  In the first, you have your email address is @gmail.com.  The second is Gmail for domains.  The first is a little easier, due to the way that Google has generated their SSL/TLS certificates.

The important piece of information that is required is the host and port name to connect to on the network.  In Jabber-Net terms, this is the `NetworkHost`.  You can find this information from the DNS.  Later versions of Jabber-Net, on Windows, will do this DNS lookup automatically.  If you want to perform the lookup manually, you can use the `nslookup` command:

```
C:\>nslookup
Default Server:  [my_dns_server]
Address:  [my_dns_server_ip]

> set q=SRV
> set noverbose
> _xmpp-client._tcp.gmail.com.
Server:  [my_dns_server]
Address:  [my_dns_server_ip]

Non-authoritative answer:
_xmpp-client._tcp.gmail.com     SRV service location:
          priority       = 5
          weight         = 0
          port           = 5222
          svr hostname   = talk.l.google.com
_xmpp-client._tcp.gmail.com     SRV service location:
          priority       = 20
          weight         = 0
          port           = 5222
          svr hostname   = talk1.l.google.com
_xmpp-client._tcp.gmail.com     SRV service location:
          priority       = 20
          weight         = 0
          port           = 5222
          svr hostname   = talk2.l.google.com
_xmpp-client._tcp.gmail.com     SRV service location:
          priority       = 20
          weight         = 0
          port           = 5222
          svr hostname   = talk3.l.google.com
_xmpp-client._tcp.gmail.com     SRV service location:
          priority       = 20
          weight         = 0
          port           = 5222
          svr hostname   = talk4.l.google.com

```

# gmail.com JIDs #

  * If you're on Windows, and on a late-model version of Jabber-Net, it should "just work".
```
JabberClient jc = new JabberClient();
jc.User = "username";   // just the username, not including the @domain.
jc.Server = "gmail.com";
jc.Password = "password";
```

  * If you're on Mono, or an earlier version of Jabber-Net, you'll need to set the `NetworkHost` property:
```
jc.NetworkHost = "talk.l.google.com";  // Note: that's an "L", not a "1".
```

# Google for Domains #

## Turn on Chat ##

From your mail page, go to "Manage this domain", then "Chat", then make sure it says "Disable Chat" at the bottom.  You obviously must be an administrator of your domain to do this.

## DNS ##

If you have your own domain hosted on Google Mail, you can use the XMPP service for that domain as well.  For everything to work correctly, you'll need the following SRV records on your domain.

```
_xmpp-server._tcp.example.com. IN SRV 5 0 5269 xmpp-server.l.google.com.
_jabber._tcp.example.com. IN SRV 5 0 5269 xmpp-server.l.google.com.
_xmpp-client._tcp.example.com. IN SRV 5 0 5222 talk.l.google.com.
```

Make sure to change "example.com" to your domain name.  Don't modify the "google.com" portion.  Ask your DNS provider for help if you can't figure out how to get these set.

If you just want to get up and running quickly, don't put in the SRV records, and set the `NetworkHost` property to "talk.l.google.com". (Note that they used an "L", not a "1").

## Certificates ##

The certificate that comes on the SSL/TLS handshake will be invalid, in that the name will not match your domain.  The best way to deal with this is to catch the OnInvalidCertificate event.

```
jc.OnInvalidCertificate += new System.Net.Security.RemoteCertificateValidationCallback(jc_OnInvalidCertificate);

...
bool jc_OnInvalidCertificate(object sender,
                             System.Security.Cryptography.X509Certificates.X509Certificate certificate,
                             System.Security.Cryptography.X509Certificates.X509Chain chain,
                             System.Net.Security.SslPolicyErrors sslPolicyErrors)
{
    if (certificate_ok)
        return true;
    else
        return false;
}
```

The concern here is that there is a man in the middle of your conversation with Google, that is offering you a certificate with a bad name, signed by a Certificate Authority that you trust.  Take appropriate security precautions.

## Connections ##

Now, continue along as above, for gmail.com addresses, but use **your** domain name instead of gmail.com:

```
JabberClient jc = new JabberClient();
jc.User = "username";   // just the username, not including the @domain.
jc.Server = "example.com";
jc.Password = "password";
```