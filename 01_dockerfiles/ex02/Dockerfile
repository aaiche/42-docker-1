FROM ruby AS base
MAINTAINER aa <aa@aai.fr>

ENV APP_HOME /opt/app

# 
# Installer utilities et clean up: objectif avoir l image la plus ptite
# D apres openclassroom:
#   - Node.js est un prérequis pour le bon fonctionnement de rails, il intervient dans la gestion des fichiers JavaScript de vos futurs sites
#

#apt-get install -qqy build-essential libpq-dev libffi-dev libssl-dev libreadline-dev zlib1g-dev libsqlite3-dev && \
RUN apt-get update -qqy                                     \
 && apt-get install -qqy build-essential libpq-dev nodejs   \
 && rm -rf /var/lib/apt/lists/*                             \
 && apt-get autoremove -y                                   \
 && apt-get clean 

#
# Installer Rails
# D apres openclassroom:
#  - gem: package manager. 
#  - gem est un programme qui vient avec Ruby. 
#    Il permet d’installer d’autres programmes, comme Rails
#
# --no-document: on veut l image la plus ptite
#
RUN echo 'gem: --no-document' >> ~/.gemrc   \
 && gem install rails

#
# Versions:
#
RUN ruby -v     \
 && gem -v      \
 && bundler -v  \
 && rails -v


#EXPOSE 3000
#CMD ["rails", "server", "--binding", "0.0.0.0”, "-p", "3000"] 

#
# Cette image servira a generer une app ruby on rails. Utiliser le script ./build-and-run-Fille.sh affiche a la fin
# Attention: ici je m amuse, je ne pense pas que ce genre ait sa place dans un Dockerfile
# Decommenter si on veut une app ruby on rails
FROM base AS example
RUN ["rails", "new", "weblog"]
WORKDIR "weblog"
#RUN "bash -c 'cd /weblog'"
RUN ["rails", "generate", "scaffold", "post", "title:string", "body:text"]
#RUN ["rails", "db:migrate"]
RUN mkdir Fille
RUN echo 'FROM ft-rails:on-build'                                                  > ./Fille/Dockerfile
RUN echo 'EXPOSE 3000'                                                             >> ./Fille/Dockerfile
RUN echo 'CMD ["rails", "server", "--binding", "0.0.0.0", "-p" ,"3000"]'           >> ./Fille/Dockerfile
RUN touch ./build-and-run-Fille.sh                                                                             \
 && chmod +x ./build-and-run-Fille.sh                                                                          \
 && echo '#! /bin/bash'                                                            >> ./build-and-run-Fille.sh \
 && echo ': ${1?"Usage: $0 <image_ID> --> image_ID est l id de l image example "}' >> ./build-and-run-Fille.sh \
 && echo 'image_id=$1'                                                             >> ./build-and-run-Fille.sh \
 && echo 'mkdir Fille.sav'                                                         >> ./build-and-run-Fille.sh \
 && echo 'docker container run --detach --name example $image_id sleep 1d'         >> ./build-and-run-Fille.sh \
 && echo 'docker container cp example:/weblog/ ./Fille.sav'                        >> ./build-and-run-Fille.sh \
 && echo 'docker container cp example:/weblog/Fille/Dockerfile ./Fille.sav'        >> ./build-and-run-Fille.sh \
 && echo 'mkdir Fille'                                                             >> ./build-and-run-Fille.sh \
 && echo 'cp Fille.sav/Dockerfile ./Fille'                                         >> ./build-and-run-Fille.sh \
 && echo 'cp -r Fille.sav/weblog/* ./Fille/'                                       >> ./build-and-run-Fille.sh \
 && echo 'cd ./Fille'                                                              >> ./build-and-run-Fille.sh \
 && echo 'docker image build --tag=fille  . '                                      >> ./build-and-run-Fille.sh \
 && echo 'docker container run --detach --name=c2 -p 4242:3000 fille'              >> ./build-and-run-Fille.sh \
 && echo '# http://<ip>:4242/posts'                                                >> ./build-and-run-Fille.sh \
 && cat ./build-and-run-Fille.sh                                       



FROM base AS ft-rails
#
# ONBUILD: 
# - Appelee trigger
# - On construit une image de base Mere avec Dockerfile contenant des instructions ONBUILD
# - Ces instructions ne sont pas executees dans cette image de base Mere
# - Elles le seront lorsque des images Filles seront creees a partir de cette image de base
ONBUILD RUN mkdir $APP_HOME

#
# Copier ces fichiers a part, car on utilisera 'bundle install' qu une seule fois.
#
ONBUIlD COPY Gemfile Gemfile.lock $APP_HOME/

ONBUILD WORKDIR $APP_HOME

#
# Installer Rails
# D apres openclassroom:
#  - Les gems sont des modules de codes produits par d'autres développeurs qui apportent des fonctionnalités à votre application Ruby – elles ne se limitent pas à Rails. 
#    Rails utilise lui-même quelques gems. 
#    La liste des gems utilisées se situe dans le fichier 'Gemfile' à la racine de votre application.
#  - Lorsque l’on ajoute une gem au Gemfile, il faut la télécharger et l’installer. 
#    Ces deux choses se font automatiquement avec la commande 'bundle install', à faire dans le terminal. 
#    Les gems sont alors téléchargées sur le site indiqué en toute première ligne du 'Gemfile', la source. Cette ligne ne change presque jamais.
#  - rake: est une boite à outils. Elle permet de faire plein de choses, notamment d’exécuter les migrations pour qu’elles soient prises en compte. 
#  - Pour les applications Rails, après le 'bundle install', il ne faut pas oublier:
#      + rake db:create (parfois facultatif), puis
#      + rake db:migrate.
#
ONBUILD RUN bundle install


#
# Copier le code de l app
#
ONBUILD COPY . $APP_HOME

ONBUILD RUN rake db:migrate
#ONBUILD RUN rake db:schema:load
ONBUILD RUN rake db:setup





#
# build l image mere: ft-rails:on-build:
#
#   docker image build --tag=ft-rails:on-build .
#

# Run
# executer le script Fille.. il est affiche lors du build
#
#--#! /bin/bash
#--: ${1?"Usage: $0 <image_ID> --> image_ID est l id de l image example "}
#--image_id=$1
#--mkdir Fille.sav
#--docker container run --detach --name example $image_id sleep 1d
#--docker container cp example:/weblog/ ./Fille.sav
#--docker container cp example:/weblog/Fille/Dockerfile ./Fille.sav
#--mkdir Fille
#--cp Fille.sav/Dockerfile ./Fille
#--cp -r Fille.sav/weblog/* ./Fille/
#--cd ./Fille
#--docker image build --tag=fille  .
#--docker container run --detach --name=c2 -p 4242:3000 fille
#--# http://<ip>:4242/posts
