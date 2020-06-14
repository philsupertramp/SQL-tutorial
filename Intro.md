# database setup for local development

1) install docker and docker-compose
2) start database server (this might take some time)
`docker-compose up -d`
[Optional] 3) open terminal for logs `docker-compose logs -f db`
4) log into postgres-shell
`docker-compose exec db psql -U postgres`
5) learn SQL

