FROM node

WORKDIR /usr/src/app
#
# installer
#
RUN curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version 0.26.1
RUN yarn global add nodemon@1.11.0
RUN apt-get update                  \
 && apt-get upgrade -y              \
 && apt-get install -yqq vim        \
 && apt-get install -yqq net-tools  
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#
# mkdir working directory du container
#
RUN mkdir -p /usr/src/app

#
# copier host local yarn cache  ...
#
ADD .yarn_cache /usr/local/share/.cache/yarn/v1/

#
# build les dependances et creer un lien vers les modules
# qu on met dans /tmp/ car on va utiliser un mount bind
#
ADD ./package.json ./yarn.* /tmp/
#RUN mkdir -p /usr/local/share/.cache/yarn/v1.sav
#RUN cp -r /usr/local/share/.cache/yarn/v1/* /usr/local/share/.cache/yarn/v1.sav/
RUN cd /tmp/ && yarn
RUN cp -r /tmp/node_modules .

#
# on ajoute le code source
#
ADD . /usr/src/app/

EXPOSE 3000 


ENTRYPOINT ["sh", "-c"]
CMD ["nodemon", "server.js"]

#CMD ["cp -r /tmp/node_modues . & cp /tmp/yarn.lock yarn.lock & nodemon server.js  & if [ '(tar -cf - /usr/local/share/.cache/yarn/v1 | crc32)' != '(tar -cf - .yarn_cache | crc32)' ]; then cp -r /usr/local/share/.cache/yarn/v1/. .yarn_cache/; fi;"]

#
# TBD
#
#   - Gerer proprement le cache de yarn: JE CROIS QUE C EST LE BUT DE L EXERCICE:
#     cf. https://medium.com/@shemleong/using-docker-and-yarn-for-development-2546e567ad2
#   - Bind Mount le repertoire source avec le container ie: --volume $(pwd):/usr/src/app
#   - Verifier si l on a besoin d installer 'yarn', car je pense qu il est installe par defaut maintenant
#   - Avec cette methode : que se passe t il si je modifie le code source et ai besoin de yarn'er un nouveau package
#


#
# Build:
#
#   docker build -t aa .
#
# Run:
#
#	mkdir .yarn_cache/
#
#   docker rm -f c1; docker run -d --name c1 -p 3000:3000 aa; docker ps -a
#  
#   puis:   http://192.168.99.100:3000
#
# Modifier le code source:
#
#   docker exec -it c1 vim .
#
