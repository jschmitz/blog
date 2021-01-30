---
layout: post
title: Five Tips for Troubleshooting Docker, Rails and Postgres Setup
date: 2021-01-30 06:01 -0600
---
These five tips will help you if you run into trouble setting up your Rails app and your database using Postgres as an example. The setup to connect two containers, a application container and database container can become a challenge if your config is not quite right and you may start seeing errors like "No Postgres password supplied" or "TypeError (no implicit conversion of nil into String)" and this post can help!

With a few troubleshooting tips and tricks setting up your Rails application to run with Docker and containerized Postgres image can be a quick process.

First, lets check out what out Dockerfile, docker-compose.yml and config/database.yml will look like when completed.

## Dockerfile
```YAML
# Dockerfile
# Use ruby image to build our own image
FROM ruby:2.7

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

WORKDIR /usr/src/app
# We copy these files from our current application to the /app container
COPY Gemfile Gemfile.lock ./
RUN bundle install

COPY . .
CMD ["rails", "server", "-b", "0.0.0.0"]
```

## docker-compose.yml
```YAML
# docker-compose.yml
version: '3.0'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${APP_DB_USER}
      POSTGRES_PASSWORD: ${APP_DB_PASSWORD}
      POSTGRES_DB: ${APP_DB_USER}
  web:
    build: .
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - APP_DB_HOST=${APP_DB_HOST}
      - APP_DB_PASSWORD=${APP_DB_PASSWORD}
      - APP_DB_USER=${APP_DB_USER}
```

## config/database.yml
```YAML
# config/database.ymlk
default: &default
  adapter: postgresql
  encoding: unicode
  host: <%= ENV["APP_DB_HOST"] %>
  username: <%= ENV["APP_DB_USER"] %>
  password: <%= ENV["APP_DB_PASSWORD"] %>
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
```

# 1. Read the DockerHub Postgres instructions
I know weird... reading the instructions. In this case I have a suspicion this will save you time in your setup. Specifically check out the environment variables that you can set when initializing your Postgres instance.

With that, click the link! [DockerHub Postgres](https://hub.docker.com/_/postgres/)

# 2. Use environment variables
The absolute worst part of setting up Docker and Docker Compose for your app is rebuilding your image multiple times. Using environment variables out of the gate for your Postgres image initialization and your database.yml will help you to minimize those rebuilds.

The work for you to do for this is to update your database.yml file to use environment variables. This will save you a bit of time in your setup and give you more flexibility when running your application.


# 3. Delete the Postgres data directory to update
Let's check out the db section of the Docker Compose more closely, specifically the initialization of the DB. The Postgres database is initialized once. The User, Password and DB (password is the minimum input required) is initialized once the data directory is populated. The data directory is defined in the volume attribute, in this case your local directory will be ./tmp/db.

```YAML
version: '3.0'
services:
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${APP_DB_USER}
      POSTGRES_PASSWORD: ${APP_DB_PASSWORD}
      POSTGRES_DB: ${APP_DB_USER}
```

To update your Postgres, delete the data director for Postgres to so that on the next startup the Postgres will reinitialize using your updated environment variables.

```bash
rm -rf tmp/db/
```

You can change your environment variables all you want and that will not help you out. Postgres and its initial user is NOT reinitialized or update if you change this section

# 4. Check Postgres has initialized as expected
A couple of commands will help you to verify that Postgres is running as expected and here is a quick way to do so.

Check that your container image is running:
```bash
docker ps

CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS         PORTS                    NAMES
b03bab16dbc6   APP_web           "rails server -b 0.0…"   10 seconds ago   Up 8 seconds   0.0.0.0:3000->3000/tcp   APP_web_1
c2b8328d0935   postgres          "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   5432/tcp                 APP_db_1
1a3583832166   postgres:latest   "docker-entrypoint.s…"   24 hours ago     Up 24 hours    5432/tcp                 somestackname_db.1.dm1dinndadduvk91rirahcgvu
```

Use the container image name to open a shell
```bash
docker exec -it APP_db_1 /bin/bash
```

Once in the shell access psql
```bash
psql -U <your initial pg user>
```

Once you've entered the psql you'll be able verify what has been initialized. Here's a couple of commands to help verify the setup:

| psql Command        | Description        |
| ---------------   | ---------------    |
| \du               | Show all Users     |
| \l                | Show databases     |
| \c <databasename> | Switch to database |
| \q                | quit               |

# 5. Use environment variables again
Using the local environment variables of your system will takes a tiny bit of setup time and will save you a good deal of time in the long run. This will help to ensure your Postgres credentials and db match your application database expectations and provide you with good flexibility to re-use your containers.

```YAML
# docker-compose.yml
environment:
  POSTGRES_USER: ${APP_DB_USER}
  POSTGRES_PASSWORD: ${APP_DB_PASSWORD}
  POSTGRES_DB: ${APP_DB_USER}
```

```YAML
# config/database.yml
default: &default
  adapter: postgresql
  encoding: unicode
  host: <%= ENV["APP_DB_HOST"] %>
  username: <%= ENV["APP_DB_USER"] %>
  password: <%= ENV["APP_DB_PASSWORD"] %>
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
```

# References
* https://medium.com/@guillaumeocculy/setting-up-rails-6-with-postgresql-webpack-on-docker-a51c1044f0e4
* https://docs.docker.com/compose/rails/
* https://hub.docker.com/_/postgres/
* https://vsupalov.com/difference-docker-compose-and-docker-stack/
* https://www.postgresqltutorial.com/postgresql-show-tables/
