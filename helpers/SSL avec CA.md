http://webdevpro.net/https-mise-en-place-dun-certicat-auto-signe/

## Création d’un dossier dédié :
```
mkdir /etc/ssl/private/
cd /etc/ssl/private/
```
## générer la clé privée du serveur et la protéger avec un chmod 400.
```
openssl genrsa -out gitlab.example.com.key 2048
chmod 400 gitlab.example.com.key
```
## demande de signature certificat du serveur :
A partir de gitlab.example.com.key (votre clé SSL Privée), nous allons créer un fichier de demande de signature de certificat (CSR Certificate Signing Request) pour notre serveur :

```
openssl req -new -key gitlab.example.com.key -out gitlab.example.com.csr
```

La commande génère le fichier gitlab.example.com.csr. Ce fichier contient la clé publique à certifier.
Maintenant, deux choix s’offrent à nous :

envoyer le fichier gitlab.example.com.csr à un organisme (le tiers de confiance ou l’autorité de certification (AC) comme VeriSign ou Comodo aussi appelés Public Key Infrastructure (PKI)) et ainsi obtenir le certificat dûment signé par la clé privée de l’organisme (après avoir payé)
ou bien signer vous-même le certificat.
C’est le deuxième cas que nous allons traiter ici.

## clé privée du l’autorité de certification (AC) :
Pour signer un certificat, nous devons devenir notre propre autorité de certification, cela implique donc de générer une clé (ca.key) et un certificat auto-signé (ca.crt).
```
openssl genrsa -out ca.key 2048
chmod 400 ca.key
```
La commande génère le fichier ca.key : clé SSL privée de notre AC.

## certificat autosigné de l’autorité de certification (AC) :
A partir de ca.key (clé SSL de l’AC), nous créons un certificat x509 pour une durée de validité de 10 ans auto-signé :

```
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
```
> Attention : le nom de « Common Name » doit être différent de celui qui a été donné précédemment.




# signature du certificat serveur par le CA :
Enfin la commande suivante signe la demande de certificat :
```
openssl x509 -req -in gitlab.example.com.csr -out gitlab.example.com.crt -CA ca.crt -CAkey ca.key -CAcreateserial -days 365
```
