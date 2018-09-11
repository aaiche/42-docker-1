# BYOTSS = Build Your Own TeamSpeak Server
# +-------------------------------------+
# |                                     |
# |  +---------------------------+      |
# |  |C:debian+ts3               |      |
# |  |                           |      |
# |  |                           |      |
# |  |                           |      |
# |  |                           |      |
# |  +-+----------------+-----+------+  |
# |  | | Docker Engine: |     |      |  |
# |  | | 'default'      |     |      |  |
# |  | |                |     |      |  |
# |  | v                v     v      |  |
# |  |9987           10011  30033    |  |
# |  | |                |     |      |  |
# |  +-v-- + -----------v-----v------+  |
# |    |   |            |     |         |
# |    | IP: 192.168.99.100   |         |
# |    |   |            |     |         |
# |    |   v            |     |         |
# |    |                v     |         |
# |    |             10011    |         |
# |   9987              |     v         |
# |    |                |   30033       |
# |    v                v     |         |
# |                           v         |
# +-------------------------------------+


FROM debian
MAINTAINER aa <aa@aai.fr>

#
# Contraintes: teamSpeak3 Client de MSC est en version 3.0.18.2
#   ENV TS_RELEASE=3.3.1        ==> on ne peut pas, car demande d upgrade teamrSpeak3 Client
#   ENV TS_RELEASE=3.0.18.2 ==> problemes de untar et un'b'zip de l archive
ENV TS_RELEASE=3.0.13
ENV TS_USER_HOME_DIRECTORY=/home/teamspeak
ENV TS_DIRECTORY=${TS_USER_HOME_DIRECTORY}/app
ENV TS_USER=teamspeak


# 
# Installer utilities et clean up: objectif avoir l image la plus ptite
#   procps: pour avoir la commande ps: voir TBD CTRL C
#
RUN apt-get update -qy &&                               \
    apt-get install -qy wget tar bzip2 &&               \
    apt-get install -y procps   &&                      \
    rm -rf /var/lib/apt/lists/* &&                      \
    apt-get autoremove -y &&                            \
    apt-get clean 

# 
# Creer groupe et user ${TS_USER}, teamSpeak3 Server ne doit pas etre demarre en tant que root
#
RUN groupadd -r ${TS_USER} &&                                               \
    useradd -r -m -g ${TS_USER} -d ${TS_USER_HOME_DIRECTORY} ${TS_USER} &&  \
    mkdir -p ${TS_DIRECTORY} &&                                             \
    chown -R ${TS_USER}: ${TS_DIRECTORY} 

# 
# Telecharger archive teamspeak3 et installer 
#
RUN wget --quiet --output-document=/teamspeak.tar.bz2  http://dl.4players.de/ts/releases/${TS_RELEASE}/teamspeak3-server_linux_amd64-${TS_RELEASE}.tar.bz2 && \
    tar -xjf /teamspeak.tar.bz2 &&  \
    mv teamspeak3-server_linux_amd64/* ${TS_DIRECTORY} && \
    rm -rf /teamspeak.tar.bz2 /teamspeak3-server_linux_amd64/



#  9987 default voice
# 10011 server query
# 30033 file transport
EXPOSE 9987/udp 10011 30033 

WORKDIR ${TS_DIRECTORY}

ENTRYPOINT ["bash", "ts3server_minimal_runscript.sh"]

USER   teamspeak

#
# Creer image:
#
#       docker image build --tag=byotss .
#

#
# Installer via MSC client teamspeak 3
#

#
# Creer et lancer container:
#
#       docker container run --rm -p 9987:9987/udp -p 10011:10011 -p 30033:30033 --name=c2 byotss
#
#

# 
# Tester:
#
#       Lancer via Launchpad "TeamSpeak 3 client"
#       Connect
#       @ip du docker engine
#       entrer le toker donne par 'docker container run ....'
#

#
# TBD: 
#   - il faut pouvoir lancer le script en PID=1 afin qu il puisse recevoir ^C
#   - faire en sorte que ttes les donnees soient dans un volume separe
# RUN ./ts3server_minimal_runscript.sh createinifile=1
# RUN mkdir /data && chown teamspeak:teamspeak /data
# VOLUME ["/data"]
#   RUN echo "license_accepted=1" >  .ts3server_license_accepted ==> Ceci est necessaire pour les dernieres versions:
#

