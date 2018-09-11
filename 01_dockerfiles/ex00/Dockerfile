FROM alpine
MAINTAINER aa <aa@aai.fr>

RUN apk update \
 && apk add vim \
 && rm -rf /var/cache/apk/*


CMD ["/usr/bin/vim"]


#
# build image:
#           docker image build --tag=alpine_vim .
#
# run container:
#           docker container run --rm --name=c1 -it alpine_vim
#
#           docker container run --name=c1 -it alpine_vim
#			docker container start ...
#			docker container exec -it ... ash
#
# remove container 
#           docker container rm -f c1
#


#
# TBD:
#	+  a tester:  apk --no-cache add
#
