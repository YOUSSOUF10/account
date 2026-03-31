

# Building
- Java 8
- plugin lombok 
- $ docker-compose -f docker/docker-compose.yml up -d  (for mongoDB)

## Launch tests

$ mvn clean install


FROM python:3.11

WORKDIR /app

COPY requirements.txt /tmp/requirements.txt
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    libreoffice \
 && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir -r /tmp/requirements.txt

COPY . .
RUN chmod +x docker/entrypoint.sh

VOLUME ["/var/log"]

EXPOSE 8080

CMD ["./docker/entrypoint.sh"]
