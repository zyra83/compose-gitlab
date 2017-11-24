Premièrement les certificats c'est désormais nom de domaine.key/crt.
Et c'est plus dans certs, c'est dans /etc/gitlab/ssl

Suivre les 4 étapes de SSL dans https://github.com/sameersbn/docker-gitlab



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
