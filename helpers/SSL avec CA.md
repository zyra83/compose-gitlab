# HOW TO HTTPS

# TL;DR;
``` bash
# On va avoir besoin de ce fichier par la suite
cat << EOF > ma_config_openssl.cnf 
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt = no
default_md = sha256

[ req_distinguished_name ]
countryName                = FR
stateOrProvinceName        = Landes
localityName               = Geloux
organizationName           = FlashCorp.
commonName                 = gitlab.example.com

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1   = gitlab.example.com
DNS.2   = pages.gitlab.example.com
EOF
# génération des DH au besoin
openssl dhparam -out dhparam.pem 2048
# clé privée du CA
openssl genrsa -out "ca.key" 2048
chmod 400 "ca.key"
# autosignature du CA
openssl req -new -x509 -days 3650 -key "ca.key" -out "ca.crt" -subj "/C=FR/ST=Landes/L=Geloux/O=Flash Corp./OU=IT Department/CN=Flash Corp. Certification Autority"
# clé privée du serveur
openssl genrsa -out "gitlab.example.com.key" 2048
chmod 400 "gitlab.example.com.key"
# Demande de signature (csr) version non-interactive
openssl req -new -key "gitlab.example.com.key" -out "gitlab.example.com.csr" -config ma_config_openssl.cnf
# signature du CSR par le CA (et rappel de config, bug openssl ? )
openssl x509 -req -in "gitlab.example.com.csr" -out "gitlab.example.com.crt" -CA "ca.crt" -CAkey "ca.key" -CAcreateserial -days 365 -extensions v3_req -extfile ma_config_openssl.cnf 
```

http://webdevpro.net/https-mise-en-place-dun-certicat-auto-signe/
# Lexique

- http://www.gtopia.org/blog/2010/02/der-vs-crt-vs-cer-vs-pem-certificates/
- https://fr.wikipedia.org/wiki/Demande_de_signature_de_certificat
- https://serverfault.com/questions/845806/how-to-issue-ssl-certificate-with-san-extension
- https://gist.github.com/bradmontgomery/6487319
- https://www.digicert.com/csr-ssl-installation/nginx-openssl.htm


**Certificate Signing Request** : extension .csr, fichier de demande de signature d'un certificat. Il donne lieu à la création d'un .c(e)rt une fois signé par l'autorité de certification


## Création d’un dossier dédié :
```
mkdir "/etc/ssl/"
cd "/etc/ssl/"
```

## 1. Clé privée de l’autorité de certification (AC)

Pour signer un certificat, nous devons devenir notre propre autorité de certification, cela implique donc de générer une clé (ca.key) et un certificat auto-signé (ca.crt).

```
openssl genrsa -out "ca.key" 2048
chmod 400 "ca.key"
```

La commande génère le fichier ca.key : clé SSL privée de notre autorité de certification.

## 2. certificat autosigné de l’autorité de certification (AC) :
A partir de ca.key (clé SSL de l’AC), nous créons un certificat x509 pour une durée de validité de 10 ans auto-signé :

``` bash
# Certificat auto-signé version non-interactive
openssl req -new -x509 -days 3650 -key "ca.key" -out "ca.crt"
# Certificat auto-signé version non-interactive
openssl req -new -x509 -days 3650 -key "ca.key" -out "ca.crt" -subj "/C=FR/ST=Landes/L=Geloux/O=Flash Corp./OU=IT Department/CN=Flash Corp. Certification Autority (FCCA \o/)"
```
> Attention : le nom de « Common Name » doit être différent de celui qui sera donnée aux certificats que l'on va signer avec ce CA.

## 3. Clé privée du serveur
Création de la clé privée du serveur (server.key), elle est unique et ne doit jamais être révélée à quiconque. 
On change ses permissions de manière à ce qu'elle ne soit visible que de son propriétaire (lecture seule pour owner). 
``` bash
# génération d'une clé privée RSA 2048 bits
openssl genrsa -out "gitlab.example.com.key" 2048
# lecture seule pour owner, innaccessible à group et others
chmod 400 "gitlab.example.com.key"
```
## 4. demande de signature certificat du serveur :

A partir de gitlab.example.com.key (votre clé SSL Privée), nous allons créer un fichier de demande de signature de certificat (CSR Certificate Signing Request) pour notre serveur :

``` bash
# Demande de signature (csr) version interactive
openssl req -new -key "gitlab.example.com.key" -out "gitlab.example.com.csr"
# Demande de signature (csr) version non-interactive
openssl req -new -key "gitlab.example.com.key" -out "gitlab.example.com.csr" -subj "/C=FR/ST=Landes/L=Geloux/O=Flash Corp./OU=IT Department/CN=gitlab.example.com"

```

La commande génère le fichier gitlab.example.com.csr. Ce fichier contient la clé publique à certifier.
Maintenant, deux choix s’offrent à nous :

envoyer le fichier gitlab.example.com.csr à un organisme (le tiers de confiance ou l’autorité de certification (AC) comme VeriSign ou Comodo aussi appelés Public Key Infrastructure (PKI)) et ainsi obtenir le certificat dûment signé par la clé privée de l’organisme (après avoir payé)
ou bien signer vous-même le certificat.
C’est le deuxième cas que nous allons traiter ici.

# 5. signature du certificat serveur par le CA :
Enfin la commande suivante signe la demande de certificat :

``` bash
openssl x509 -req -in "gitlab.example.com.csr" -out "gitlab.example.com.crt" -CA "ca.crt" -CAkey "ca.key" -CAcreateserial -days 365
```
