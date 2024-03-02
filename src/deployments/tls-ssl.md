# Setting up TLS/SSL
To set up TLS/SSL is fairly straightforward. We will be using LetsEncrypt for this, which is completely free to use. However, if you want to use them there are also alternative services that offer TLS certs but will charge money for it.

## Side-note
Most places that use dockerfile deployments as the primary form of deployment will typically manage SSL for you. However if you're not sure, it's always best to ask (or deploy a service and check for yourself)!

## Setting up LetsEncrypt
LetsEncrypt has a guide which you can follow [here](https://letsencrypt.org/getting-started/). It primarily suggests using Certbot, a CLI tool you can use for entirely free to either completely automate away SSL or just get a certificate.
