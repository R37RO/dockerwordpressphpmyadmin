# Wordpress docker
docker nginx webserver
with docker-compose with mysql:8.0, wordpress:6.0-fpm-alpine, nginx:1.23.1-alpine, phpmyadmin/phpmyadmin

```yml
version: '3'

services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env #rename hidden.env file to .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes: 
      - dbdata:/var/lib/mysql
    expose:
      - "3306"
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on: 
      - db
    image: wordpress:6.0-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env #rename hidden.env file to .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini

    networks:
      - app-network

  webserver:
    depends_on:
      - wordpress
    image: nginx:1.23.1-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "8001:80"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d

    networks:
      - app-network

  # if you want phpMyAdmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadminn
    restart: unless-stopped
    depends_on:
      - db
    ports:      
      - "8080:80"        
    environment:
      PMA_HOST: db
      PMA_USER: ${MYSQL_USER}
      PMA_PASSWORD: ${MYSQL_PASSWORD} 
      UPLOAD_LIMIT: 64M
    networks:
      - app-network

volumes:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge
```
