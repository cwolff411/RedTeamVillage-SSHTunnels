# Installing Stunnel
Stunnel needs to be installed on the server and client. It's included in most if not all, package managers or you can always download a tar from the [Stunnel](https://stunnel.org) website.

If you download it from the package manager a default SSL certificate will be included in the `/etc/stunnel/` folder. If you download the tar archive you can use `make` to generate certificates.

You can run Stunnel as a daemon or as a single binary. Either way you need to create a `stunnel.conf` file with your configuration.

In this folder you'll find configuration files for an Stunnel client and server. You'll notice that they're very similar with the exception being that the client configuration only requires the private key for the certificate.

These config files setup an SSL tunnel between the client and server on port `443` and listens on the clients localhost port `2222`.

To use this tunnel simply ssh using port `2222` and it will be routed over the SSL tunnel on port `443` to the server. The server then receives it on 443 and redirects it to `22`.