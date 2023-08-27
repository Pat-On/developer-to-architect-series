# Docker network and volumes

```
version: '3.5'

networks:
  mynet1:
    name: mynet1

volumes:
  postgres-data:
    driver: local
  postgres-logs:
    driver: local
  app-logs:
    driver: local


```

## Example of services
```
services:

  web:
    build:
      context: ../../web
      dockerfile: Dockerfile
    image: ntw/web
    container_name: web
    hostname: web
    networks:
      - mynet1                    <------
    ports:
      - "8000:8000"
    command: python3 manage.py runserver 0.0.0.0:8000
    env_file: .env
    volumes:
      - app-logs:/var/log/oms   <------

      
  services:
    build:
      context: ../../services
      dockerfile: Dockerfile
    image: ntw/services
    container_name: services-build

  admin-svc:
    image: ntw/services
    container_name: admin-svc
    hostname: admin-svc
    networks:
      - mynet1                    <------
    ports:
      - "8081:8080"
    command: admin
    env_file: .env
    volumes:
      - app-logs:/var/log/oms   <------

```