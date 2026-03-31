

# Building
- Java 8
- plugin lombok 
- $ docker-compose -f docker/docker-compose.yml up -d  (for mongoDB)

## Launch tests

$ mvn clean install


FROM docker.artifactory-dogen.group.echonet.net.intra/python:3.11

WORKDIR /app

COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

COPY . .
RUN chmod +x docker/entrypoint.sh

VOLUME ["/var/log"]

EXPOSE 8080

CMD ["./docker/entrypoint.sh"]
