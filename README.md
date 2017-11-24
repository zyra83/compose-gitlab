# activer https sur le gitlab
Premièrement les certificats c'est désormais nom de domaine.key/crt.
Et c'est plus dans certs, c'est dans /etc/gitlab/ssl

Suivre la procédure dans helpers SSL + CA

docker exec -it gitlab_web_1 /bin/bash
cd /etc/gitlab/ssl/
openssl genrsa -out gitlab.example.com.key 2048
chmod 400 gitlab.example.com.key
openssl req -new -key gitlab.example.com.key -out gitlab.example.com.csr
// pour le FQDN mettre *.example.com, le reste on s'en fout
openssl x509 -req -in gitlab.example.com.csr -out gitlab.example.com.crt -CA ca.crt -CAkey ca.key -CAcreateserial -days 365
gitlab-ctl restart

# ajouter le gitlab-runner avec le CA perso
https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/configuration/advanced-configuration.md
shared peut être mis à zero pour un illimité sur les builds conccurents.
Il semble qu'il faille obligatoirement passer par l'ihm de gitlab pour définir si un runner doit être partagé en non-interactif.

gitlab-runner register --non-interactive \
    --url "https://gitlab.example.com/" \
    --registration-token "Xj5oaipKzz-Ak--H7jh3" \
    --tls-ca-file "/home/mickael/Bureau/Gitlab/gitlab/config/ssl/ca.crt" \
    --executor "docker" \
    --name "z68-runner" \
    --limit 3 \
    --docker-image "ubuntu" \
    --docker-privileged "true" \
    --docker-tls_verify "false"

retourner éditer /etc/gitlab-runner/config.toml pour arrriver à ça

[[runners]]
  name = "z68-runner"
  limit = 3
  url = "https://gitlab.example.com/"
  token = "5733d3a06a7527f33dc2a56b52a535"
  tls-ca-file = "/home/mickael/Bureau/Gitlab/gitlab/config/ssl/ca.crt"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "ubuntu"
    privileged = true
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    dns = ["127.0.0.1"]
    extra_hosts = ["gitlab.example.com:192.168.1.17","pages.example.com:127.0.0.1"]
  [runners.cache]


// pour le fqdn, mettre *.pages.example.com
openssl genrsa -out pages.example.com.key 2048
chmod 400 pages.example.com.key
openssl req -new -key pages.example.com.key -out pages.example.com.csr
openssl x509 -req -in pages.example.com.csr -out pages.example.com.crt -CA ca.crt -CAkey ca.key -CAcreateserial -days 365



# activation de mattermost

Duplication des certificats gitlab.example.com en mattemrost.example.com pour éviter des configurations supplémentaires 
(spécifier le chemin du certif dans nginx mattemrost et du coup ça hébrerge le mattermost déparement du gitlab, c'est pas ce que je veux ici, c'est trop lourd pour mon serveur.
https://forum.mattermost.org/t/gitlab-mattermost-token-request-failed/824/6
à moins d'ajouter le certificat au niveau systeme ça déconne...


