FROM ubuntu:16.04
ENV TZ=Asia/Kolkata
RUN apt-get update && apt-get install -y vim build-essential git ntp curl python wget supervisor pv zip htop acl iputils-ping && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && mkdir -p /var/www/ 
RUN wget https://nodejs.org/download/release/v8.6.0/node-v8.6.0-linux-x64.tar.gz && tar -xvf node-v8.6.0-linux-x64.tar.gz && mv node-v8.6.0-linux-x64 /usr/local/ && ln -s /usr/local/node-v8.6.0-linux-x64/bin/node /usr/bin/node && ln -s /usr/local/node-v8.6.0-linux-x64/bin/npm /usr/bin/npm

EXPOSE 80


#npm install jscpd -g
RUN sed -i '/\[supervisord\]/a nodaemon=true' /etc/supervisor/supervisord.conf
CMD ["/usr/bin/supervisord"]