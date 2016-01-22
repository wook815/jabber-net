In Mono, you get this error:

```
ERROR: System.IO.IOException: The authentication or decryption has failed. ---> Mono.Security.Protocol.Tls.TlsException: Invalid certificate received form server.
```

This means that your server has an SSL/TLS certificate that is not signed by a Certificate Authority (CA) that your system trusts.  You can either fix the certificate problem (preferred), or set:

```
bedrock.net.AsyncSocket.UntrustedRootOK = true;
```

UntrustedRootOK is now obsolete in the head of subversion.  The default behavior is to pop up an ugly dialog box to allow the certificate.  If you want to build a better dialog box, catch OnIvalidCertificate.