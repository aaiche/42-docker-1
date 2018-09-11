# +-------------------------------------+
# |                                     |
# |  +-----------------------------+    |
# |  |C:ubuntun+gitla-ce           |    |
# |  | +           +    +          |    |
# |  | |           |    |          |    |
# |  | |           |    |          |    |
# |  | v           |    v          |    |
# |  |80           |  https:443(*) |    |
# |  |             v               |    |
# |  |         sshd:2222(*)        |    |
# |  +-+----------------+-----+----+-+  |
# |  | | Docker Engine: |     |      |  |
# |  | | 'default'      |     |      |  |
# |  | | 4GB/2CPUs      |     |      |  |
# |  | v                v     v      |  |
# |  | 80              443    22     |  |
# |  | |                |     |      |  |
# |  +-v-- + -----------v-----v------+  |
# |    |   |            |     |         |
# |    | IP: 192.168.99.100   |         |
# |    |   |            |     |         |
# |    |   v            |     |         |
# |    |                v     |         |
# |    |               443    |         |
# |   80                |     v         |
# |    |                |   2222        |
# |    v                v     |         |
# |                           v         |
# +-------------------------------------+
#
# (*): je ne comprends pas vraiment: il faut configurer ces ports 'externes' a l interieur du container
#      est ce que docker-engine ne peut pas le faire le proxy entre le container et le client
#     
#

# il n y a pas de raison particuliere d avoir choisi cette version
FROM ubuntu:16.04
MAINTAINER aa <aa@aai.fr>

ENV DOCKER_ENGINE_IP 192.168.99.101

#ENV MY_PUBLIC_ED_KEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKVkB/6QsiBEY79dtL0ZUaZYGNdPIXrzxPrkBg9sEsNt aaiche@e1r12p13.42.fr"

#
# upgrade et packages necessaires
#
RUN apt-get update                                          \
 && apt-get upgrade -y                                      \
 && apt-get install -y ca-certificates openssh-server wget  \
 && apt-get install tzdata                                  \
 && apt-get install -y postfix                              \
 && apt-get install letsencrypt -y
#tzdata: semble etre utilise pour les timezones, si pas installe: gitlab-ctl reconfigure --> echec

#
# telecharger gitlab et lancer son installation
# script.deb.sh va encore installer des packages
#
RUN wget https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh    \
 && chmod 700 script.deb.sh                                                                 \
 && ./script.deb.sh                                                                         \
 && apt-get install -y gitlab-ce                                                            \
 && rm -rf ./script.deb.sh

#
# Ces packages ne sont pas obligatoires
# Ces 2 commandes devraient etre regroupees avec les commandes ci-dessus, le but 
# etant de reduire le nbre de layers et donc la taille de l image
#
RUN apt-get update                  \
 && apt-get upgrade -y              \
 && apt-get install -yqq vim        \
 && apt-get install -yqq net-tools  \   
 && apt-get install -yqq nmap       \
 && apt-get install -yqq lsof

#
# Clean 
#
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#
# Configurer gitlab config file
# cf.  https://docs.gitlab.com/omnibus/settings/nginx.html
#
RUN sed -i "s|external_url 'http://gitlab.example.com'|external_url 'https://gitlab.example.com'|" /etc/gitlab/gitlab.rb                                \
 && sed -i "s|# nginx\['redirect_http_to_https'\] = false|nginx\['redirect_http_to_https'\] = true|" /etc/gitlab/gitlab.rb                              \
 && sed -i "s|# gitlab_rails\['gitlab_shell_ssh_port'\] = 22|gitlab_rails\['gitlab_shell_ssh_port'\] = 2222|" /etc/gitlab/gitlab.rb                         \
 && sed -i "/nginx\['redirect_http_to_https'\] = true/a  nginx['ssl_certificate'] = \"/etc/gitlab/ssl/gitlab.example.com.crt\"" /etc/gitlab/gitlab.rb   \
 && sed -i "/nginx\['redirect_http_to_https'\] = true/a  nginx['ssl_certificate_key'] = \"/etc/gitlab/ssl/gitlab.example.com.key\"" /etc/gitlab/gitlab.rb

#
# Les ports qui nous interessent
# 80: http
# 443: https
# 22: ssh
#
EXPOSE 443 80 22

#
# Creer le script de entrypoint.
# Ce ne est la facon la plus elegante ! mais on nous demande dans le sujet de ne rendre que Dockerfile !!
# cf. https://gitlab.com/gitlab-org/gitlab-ce/blob/954c1df256ac867635b31e82a032704e963a10ee/docker/assets/wrapper
#
RUN touch /usr/local/bin/wrapper.sh     \
 && chmod 700 /usr/local/bin/wrapper.sh     \
 && echo '#!/bin/bash'  >> /usr/local/bin/wrapper.sh    \
 && echo '' >> /usr/local/bin/wrapper.sh    \
 && echo 'function sigterm_handler() {' >> /usr/local/bin/wrapper.sh    \
 && echo '  echo "SIGTERM signal received, try to gracefully shutdown all services..."' >> /usr/local/bin/wrapper.sh    \
 && echo '  gitlab-ctl stop'    >> /usr/local/bin/wrapper.sh    \
 && echo '}'    >> /usr/local/bin/wrapper.sh    \
 && echo '' >> /usr/local/bin/wrapper.sh    \
 && echo 'trap "sigterm_handler; exit" TERM'    >> /usr/local/bin/wrapper.sh    \
 && echo '' >> /usr/local/bin/wrapper.sh    \
 && echo 'function entrypoint() {'  >> /usr/local/bin/wrapper.sh    \
 && echo '  gitlab-ctl stop'    >> /usr/local/bin/wrapper.sh    \
 && echo '  /opt/gitlab/embedded/bin/runsvdir-start &'  >> /usr/local/bin/wrapper.sh    \
 && echo '  gitlab-ctl reconfigure # will also start everything'    >> /usr/local/bin/wrapper.sh    \
 && echo 'sed -i "s|return 301 https://gitlab.example.com:443\$request_uri;|return 301 https://${DOCKER_ENGINE_IP}:443\$request_uri;|" /var/opt/gitlab/nginx/conf/gitlab-http.conf' >> /usr/local/bin/wrapper.sh \
 && echo "openssl rsa -in /etc/gitlab/ssl/gitlab.example.com.key -out /etc/gitlab/ssl/gitlab.example.com.key" >> /usr/local/bin/wrapper.sh \
 && echo '  gitlab-ctl restart '    >> /usr/local/bin/wrapper.sh    \
 && echo '  gitlab-ctl tail # tail all logs'    >> /usr/local/bin/wrapper.sh    \
 && echo '}'    >> /usr/local/bin/wrapper.sh    \
 && echo '' >> /usr/local/bin/wrapper.sh    \
 && echo 'entrypoint'   >> /usr/local/bin/wrapper.sh

# Pourquoi l installer dans  /usr/local/bin: parce que ce repertoire est dans le path PATH
ENTRYPOINT ["/usr/local/bin/wrapper.sh"]

#
# 1) Vm default:
#   J ai choisi d agrandir la memoire de la machine virtuelle a 2GB afin de respecter les pre-requis de gitlab
#       a) le faire avec le GUI virtualBox Mgr
#       ou
#       b) le faire en CLI via:
#           docker-machine create -d virtualbox --virtualbox-memory 2048 --virtualbox-cpu-count 2 default   
# 2) Build: 
#   docker image build --tag=my_gitlab_image .
#
# 3b) run: 
#   docker container run -it --privileged  \
#       --hostname gitlab.example.com \
#       --publish 443:443 \
#       --publish 80:80 \
#       --publish 2222:22 \
#       --name c2222 \
#       --restart always \
#       --env docker_engine_ip="192.168.99.101" \
#       my_gitlab_image
# 
# 5) Remove:
#   docker container rm -f my_gitlab_container
#

#
# Test: firefox & https
# Dans le cas ou le certificat a ete accepte: 
#   - reset firefox:
#       + Help
#       + Troubleshooting information
#       + Refresh Firefox 
# 1) GUI: http://192.168.99.100
# 2) GUI: Nouveau mot de pass
# 3) GUI: Sign in : root#nouveau mot de pass
# 4) GUI: Create projet:
#       + test 
# 5) CLI: clone le projet 'test'
#   e1r12p13% mkdir mytest
#   e1r12p13% cd mytest
#   e1r12p13% git clone https://192.168.99.100/root/test.git .
#       Cloning into '.'...
#       Username for 'https://192.168.99.100': root
#       Password for 'https://root@192.168.99.100':
#       warning: You appear to have cloned an empty repository.
#   e1r12p13% ls -a
#   ./  ../ .git/
#   e1r12p13%
# 6) CLI: gitlab push'er un fichier README.md du projet 'test'
#   e1r12p13% touch README.md
#   e1r12p13% echo "coucou" > README.md
#   e1r12p13% git add README.md
#   e1r12p13% git commit -m "add README"
#   e1r12p13% git push -u origin master
# 7) CLI: gitlab pull'er le projet 'test'
#   e1r12p13% cd /tmp
#   e1r12p13% mkdir /tmp/gitlab; cd /tmp/gitlab
#   e1r12p13% git clone https://192.168.99.100/root/test.git
#   e1r12p13% ls -a
#       ./  ../ test/
#   e1r12p13% cat test/README.md
#       coucou


#
# Test: firefox & ssh: Ne fonctionne pas
#
# 1) Entrer votre cle public dans gitlab server
#   - CLI: e1r12p13% pbcopy < ~/.ssh/id_rsa.pub
#   - GUI:
#       + A droite: icone 'sorte de globe'
#       + Settings
#       + SSH Keys
#       + Paste la cle public
#       + Add Key



#
# TBD:
#
# 1) Define data volumes: 
#   ne fonctionne pas pour moi, je veux faire du partage de fichiers avec le container (mount bind)
#       VOLUME ["/etc/gitlab"]
#       VOLUME ["/var/log/gitlab"]
#       VOLUME ["/var/opt/gitlab"]
#
# 2) ssh
#  gitlab_rails['gitlab_shell_ssh_port'] = 2222
#
# 3) Location mise en dur dans '/var/opt/gitlab/nginx/conf/gitlab-http.conf'
#   Configurer https location header
#   !!! Ce n est certainement pas comme cela qu il faut faire. 
#   !!! ne fonctionnera que si l adresse ip du docker-machine est 192.168.99.100
#   Investiguer dans les parametres proxy dans /etc/gitlab/gitlab.rc
# 
# 4) Elts d aide pour configurer:
#   - A chaque modification de '/etc/gitlab/gitlab.rc', il faut:
#       gitlab-ctl reconfigure
#       gitlab-ctl restart
#
#   - Tous les logs
#       gitlab-ctl tail
#
#   - modification des fichiers de configuration de nginx
#       gitlab-ctl hup nginx
#
#   - elts nettoyer les certificats dans firefox:
#       - reset firefox:
#           + Help
#           + Troubleshooting information
#           + Refresh Firefox 
#
#   - elts disable cache de firefox:
#           Firefox : disable cache
#           Address Bar: about:config
#           browser.cache.disk.enable --> false
#           browser.cache.memory.enable --> false
#
#   - elts pour ssh
#           + ??=> je ne m en rappelle plus!
#           + ?? e1r12p11% git config --global http.sslVerify false
#           + ?? e1r12p13% ssh-keygen -f "~/.ssh/known_hosts" -R 192.168.99.100
#           + e1r12p13% pbcopy < ~/.ssh/id_rsa.pub      ==> copier dans le buffer ma cle public
#           + e1r12p13% ssh -vvv -T -p 2222 192.168.99.101      ==> se connecter au port 2222
#           + Running sudo passwd -d git fix the problem, this can be added to the postinst script for Omnibus package of gitlab-ce, but maybe there is another solution.
# 
#           + e1r12p13% cat ~/.ssh/config
#               Host gitlab.com
#               Hostname 192.168.99.101
#               User git
#               Port 2222
#               RSAAuthentication yes
#               IdentityFile ~/.ssh/id_rsa.pub
#               PreferredAuthentications publickey



