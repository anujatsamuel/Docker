FROM ubuntu:latest
RUN apt update && apt install nginx -y 
COPY app /var/www/html
ENTRYPOINT [ "nginx", "-g", "daemon-off" ]