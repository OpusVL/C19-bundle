# EHRBase with Middleware and C19 front end

This project includes git submodules. Ensure you clone it recursively.

```shell
git clone --recurse-submodules git@github.com:OpusVL/C19-bundle.git
```

## Environment Variables

Copy the `.env.example` file to `.env` and modify as necessary.

## EHRBase

When up and running and API documented here: [http://localhost:38382/ehrbase/swagger-ui.html](http://localhost:38382/ehrbase/swagger-ui.html)

## EHRDB

Database not required to listen on external ports.

PostgreSQL data folder located at `${CONTAINER_VOLUME}/${SERIAL}/postgres` or `/srv/container/volumes/PROJECT/postgres` based on `.env` settings

## Front End

Available at [http://localhost:38380](http://localhost:38380)

## Middleware

Available at [http://localhost:38381](http://localhost:38381)