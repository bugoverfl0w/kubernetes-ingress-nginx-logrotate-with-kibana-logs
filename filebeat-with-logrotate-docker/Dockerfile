FROM docker.elastic.co/beats/filebeat:6.7.0

USER root

RUN yum clean all
RUN rm -rf /var/cache/yum/*
RUN yum install deltarpm -y
#RUN yum update -y || true
RUN yum install logrotate cronie -y 

WORKDIR /

ADD filebeat.yml /usr/share/filebeat/filebeat.yml

RUN chmod go-w /usr/share/filebeat/filebeat.yml

COPY systemctl.py /usr/bin/systemctl

RUN chmod +x /usr/bin/systemctl

RUN echo '0 * * * * > /var/log/nginx/access.log' > /var/spool/cron/root
RUN echo '1 */6 * * * > /var/log/nginx/error.log' >> /var/spool/cron/root

CMD /usr/bin/systemctl start crond 2>&1 && filebeat -e -c /usr/share/filebeat/filebeat.yml