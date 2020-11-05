# laravel-docker
Running laravel application on docker

## Organize folder

Setup laravel project folder same level as below
```
laravel-project
nginx
php-fpm
mysql
```

## Build and run application

Build images
`docker-compose build --no-cache`

Run application
`docker-compose up`

Run application at detach mode

`docker-compose up -d`

Stop application and remove containers

`docker-compose down`