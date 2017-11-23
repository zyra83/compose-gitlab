http://webdevpro.net/https-mise-en-place-dun-certicat-auto-signe/

## Création d’un dossier dédié :
```
mkdir /etc/ssl/private/
cd /etc/ssl/private/
```
## générer la clé privée du serveur et la protéger avec un chmod 400.
```
openssl genrsa -out server.key 2048
chmod 400 server.key
```
## demande de signature certificat du serveur :
A partir de server.key (votre clé SSL Privée), nous allons créer un fichier de demande de signature de certificat (CSR Certificate Signing Request) pour notre serveur :

```
openssl req -new -key server.key -out server.csr
```

La commande génère le fichier server.csr. Ce fichier contient la clé publique à certifier.
Maintenant, deux choix s’offrent à nous :

envoyer le fichier server.csr à un organisme (le tiers de confiance ou l’autorité de certification (AC) comme VeriSign ou Comodo aussi appelés Public Key Infrastructure (PKI)) et ainsi obtenir le certificat dûment signé par la clé privée de l’organisme (après avoir payé)
ou bien signer vous-même le certificat.
C’est le deuxième cas que nous allons traiter ici.

## clé privée du l’autorité de certification (AC) :
Pour signer un certificat, nous devons devenir notre propre autorité de certification, cela implique donc de générer une clé (ca.key) et un certificat auto-signé (ca.cert).
```
openssl genrsa -out ca.key 2048
chmod 400 ca.key
```
La commande génère le fichier ca.key : clé SSL privée de notre AC.

## certificat autosigné de l’autorité de certification (AC) :
A partir de ca.key (clé SSL de l’AC), nous créons un certificat x509 pour une durée de validité de 10 ans auto-signé :

```
openssl req -new -x509 -days 3650 -key ca.key -out ca.cert
```
> Attention : le nom de « Common Name » doit être différent de celui qui a été donné précédemment.




# signature du certificat serveur par le CA :
Enfin la commande suivante signe la demande de certificat :
```
openssl x509 -req -in server.csr -out server.cert -CA ca.cert -CAkey ca.key -CAcreateserial -days 365
```


```
openssl req -x509 -newkey rsa:4086 \
      -keyout key.pem -out cert.pem \
      -days 3650 -nodes -sha256
```

nano /etc/nginx/sites-enabled/
nano /etc/nginx/sites-enabled/default
cd /etc/nginx/sites-available/
cp default ciserver.gaio.org
nano ciserver.gaio.org
service nginx restart
service nginx restart journalctl -xn
journalctl -xn
cd ../sites-enabled/

rm default
rm default.BACKUP
journalctl -xn
service nginx restart
ln -s ../sites-available/ciserver.gaio.org ciserver.gaio.org
service nginx restart


root@debian-base:/etc/nginx/sites-enabled# cat /etc/nginx/sites-enabled/ciserver.gaio.org
server {
    listen *:443;
    server_name ciserver.gaio.org;
    ssl on;
    ssl_certificate                 /etc/ssl/private/server.cert;
    ssl_certificate_key    /etc/ssl/private/server.key;
}
