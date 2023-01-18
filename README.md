# example-oidc-server

Instructions on how to setup an example Apache server which uses OIDC and EGI Check-in for authentication.

This example creates a customized docker container with Apache, OIDC module and a specific configuration (client ID + client secret).

## Requirements

Assumed:
* Debian 11 server
* SSH access
* open ports 80 + 443 (HTTP + HTTPS) available at example.com

(replace example.com with your domain)

## Install and run

1. Install docker engine

see https://docs.docker.com/engine/install/debian/

2. download Apache module for OIDC

```sh
sudo apt install git
git clone https://github.com/zmartzone/mod_auth_openidc.git
cd mod_auth_openidc
```

3. obtain client ID and client secret from https://federation.rciam.grnet.gr/egi/form/new

4. adjust openidc.conf

see https://docs.egi.eu/providers/check-in/sp/ for more details

replace `CLIENT_ID_HERE` with your client ID and `CLIENT_SECRET_HERE` with your client secret from the previous step

```sh
sed -i 's/^\(OIDCOAuthVerifyJwksUri\) .*$/\1 https:\/\/aai-dev.egi.eu\/oidc\/jwk/' openidc.conf
sed -i 's/^\(OIDCOAuthRemoteUserClaim\) .*$/\1 sub/' openidc.conf
sed -i 's/^OIDCPublicKeyFiles .*$//' openidc.conf
sed -i 's/^OIDCPrivateKeyFiles .*$//' openidc.conf
sed -i 's/^OIDCCacheType .*$//' openidc.conf
sed -i 's/^OIDCRedisCacheServer .*$//' openidc.conf

sed -i 's/^\(OIDCProviderMetadataURL\) .*$/\1 https:\/\/aai-dev.egi.eu\/oidc\/.well-known\/openid-configuration/' openidc.conf
sed -i 's/^\(OIDCClientID\) .*$/\1 CLIENT_ID_HERE/' openidc.conf
sed -i 's/^\(OIDCClientSecret\) .*$/\1 CLIENT_SECRET_HERE/' openidc.conf
```

for more options, see https://github.com/zmartzone/mod_auth_openidc/blob/master/auth_openidc.conf

5. build docker container

```sh
sudo docker build -t mod_auth_openidc .
```
6. start Apache

```sh
sudo docker run -d --name apache -p 80:80 -p 443:443 mod_auth_openidc /bin/bash -c "/root/run.sh"
```

7. view logs

```sh
sudo docker logs -f apache
```

## Use

Now you can visit `https://example.com/protected/index.php` in your web browser, you will be redirected to EGI check-in to sign in,
then redirected back to a testing page which prints your OIDC attributes, access token and ID token.

You can also send an API-style request with `Authorization: Bearer <access_token>` to `https://example.com/api/index.php`:

```sh
curl -H 'Authorization: Bearer ACCESS_TOKEN_HERE' 'https://example.com/api/index.php'
```

Replace `ACCESS_TOKEN_HERE` with a valid access token (has to be fresh). You can obtain your access token for example from the testing page.
