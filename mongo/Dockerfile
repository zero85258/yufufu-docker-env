FROM mvertes/alpine-mongo

#COPY mongo.conf /usr/local/etc/mongo/mongo.conf

VOLUME /data/db /data/configdb

CMD ["mongod", "--bind_ip_all"]

EXPOSE 27017
