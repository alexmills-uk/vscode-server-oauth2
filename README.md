# VSCode secured by OAuth2

This is a relatively simple project which allows you to run an instance of vscode securely on a remote machine, which is accessed via the browser.

## How do you make it go?

Here are the TLDR instructions to "make it go":

This guide assumes that you already have VSCode server running on https://127.0.0.1:8080

* Copy `data/nginx/app.conf.example` to `data/nginx/app.conf` and substitute `example.org` for your domain.

* Copy `data/oauth2-proxy/oauth-proxy.cfg.example` to `data/oauth2-proxy/oauth-proxy.cfg` 

* Generate a cookie secret, by using something fairly random, a python example is `python3 -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'`

* [Configure the Github OIDC Provider](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/oauth_provider#github-auth-provider) by following the instructions in the oauth2-proxy docs, populate the `oauth2-proxy.cfg` file as appropriate.

* Assuming you have DNS configured for your domain, generate the initial LetsEncrypt certificates by executing `./init-letsencrypt.sh YOUR_DOMAIN_HERE`

* Start the containers with `docker compose up -d`

* Go to your domain!

## How does it actually work?

TLDR, it's a series of proxies.

Nginx terminates TLS, and sets some headers so that VSCode's websockets work correctly.

Nginx proxies to oauth2-proxy, which handles the OAuth "gumph", like getting the Authorization token, and renewing it when it expires. It also allows some filtering, to ensure only users that meet your criteria can authenticate.

Oauth2-proxy proxies to VSCode-server, which should be configured without a password. This allows you to login to your development environment remotely, using your OAuth2 credentials.