# Poc SonarQube com DotneCore
Project based on [DotNET](https://learn.microsoft.com/pt-br/dotnet/core/testing/unit-testing-with-dotnet-test) 

## Procedimento foi realizado usando o Play with Docker
http://play-with-docker.com

## Subir SonarQube usando o Docker

Exemplo abaixo não será usado, apenas para ilustrar, iremos usar o Docker Compose para facilitar a orquestração dos containers.

```sh
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e SONARQUBE_JDBC_URL=jdbc:postgresql://localhost/sonar sonarqube
```

## Sonar and Postgre

link oficial do compose.

https://github.com/SonarSource/docker-sonarqube/blob/master/recipes.md

Criar o arquivo `docker-compose.yml` e copiar e colocar as linhas abaixo.
Obs: O docker irá subir dois container, 1 com o Sonar e outro com banco de dados Postgre.

```sh
version: "2"

services:
  sonarqube:
    image: sonarqube
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    environment:
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonar
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins

  db:
    image: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - postgresql:/var/lib/postgresql
      # This needs explicit mapping due to https://github.com/docker-library/postgres/blob/4e48e3228a30763913ece952c611e5e9b95c8759/Dockerfile.template#L52
      - postgresql_data:/var/lib/postgresql/data

networks:
  sonarnet:
    driver: bridge

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_bundled-plugins:
  postgresql:
  postgresql_data:
```

## Rode o comando do Docker Compose
Esse comando irá baixar as imagens do Docker e Postgre

```sh
docker-compose up -d
```

### Exemplo de configuração
Executar esses passos no SonarQube que foi iniciado com o comando docker-compo up -d na porta 9000.

Login e senha: admin
Sonar Token: 6e284eab95abb82bf955663a2f4a2f8445fb51c3

Script que o Sonar gera para ser configurado no Pipeline
  mvn sonar:sonar \
  -Dsonar.host.url=http://ip172-18-0-3-bf4lsfk9cs9g008ekns0-9000.direct.labs.play-with-docker.com \
  -Dsonar.login=6e284eab95abb82bf955663a2f4a2f8445fb51c3

A Url e o Token do Sonar são passados via variavel.