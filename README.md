# EHRBase with Middleware and C19 front end

This repository contains a `docker-compose.yml`, which will use images from the
OpusVL Docker registry.

In order to pull from the OpusVL registry you will need to use `docker login
registry.deploy.opusvl.net`. You will need an account prior to doing so

There are several services in this bundle:

* Frontend
Image: `registry.deploy.opusvl.net/careprotect/care-protect-ui`
Github: https://github.com/OpusVL/care-protect-ui
* Middleware
Image: `registry.deploy.opusvl.net/careprotect/c19app-middleware`
Github: https://github.com/OpusVL/C19-Prototype-Middleware
* EHRBase
Image: `ehrbase/ehrbase`
Github: https://github.com/ehrbase/ehrbase
* EHRDB
Image: `ehrbase/ehrbase-postgres`
Github:(It is unclear how this is built)

## Running

To run the thing you need `docker-compose` installed on your system. Different
systems have different ways of doing that, but it's usually quite easy.

Once installed you can use `docker-compose up -d` to run it. This will pull from
the aforementioned registry, so if you are not logged in, it will fail.

### Running developer images

The images at the registry are created from other repositories, listed above. To
run a developer version of any image you will need the two pieces of information
from that list: the image name and the github URL.

    git clone <github URL>
    cd <new directory>
    docker build -t <image name> .

After the docker build process is complete, you can re-run `docker-compose up
-d` to rebuild your stack with the newly composed image.

Note that you will probably need to change branch, since the default image was
already built from master. For example, to build the middleware image on the
`xml-stuff` branch:

    git clone https://github.com/OpusVL/C19-Prototype-Middleware
    cd C19-Prototype-Middleware
    git checkout xml-stuff
    docker build -t registry.deploy.opusvl.net/careprotect/care-protect-ui .

Now your docker-compose command will bring up the new image! No guarantees are
made that the images built this way are functional. That's why the work is on a
branch.

## Environment variables

It is normal to provide the environment variables in a file called `.env`. You
will find that thi repository contains the file `.env.example`. You can copy
this to `.env` and the values within it will be picked up by the compose file.

Not every variable has a default. However, there are example values in the
`.env.example` file. You must provide values for:

; `CONTAINER_VOLUME`
: Path to a volume for postgres data
; `POSTGRES_PASSWORD`
: Password for the EHRDB postgres user
; `EHRBASE_PASSWORD`
: Password by which EHRBase connects to postgres, maybe.

You can also provide:

; `FRONTEND_IMAGE` - `registry.deploy.opusvl.net/careprotect/care-protect-ui`
: Changes the frontend image. You can use this so you don't replace your local copy of
the image at the OpusVL registry.
; `FRONTEND_IMAGE_VERSION` - `latest`
: Sometimes we might upload a tag other than `:latest`, so you can use this to
select a different one.
; `REACT_APP_STATIC_COVID` - `false`
: If you set this to `true` then the app will not use the middleware for COVID
assesments, but instead use static test data
; `REACT_APP_STATIC` - `false`
: If you set this to `true` the app will use static test data for everything,
bypassing the middleware
; `REACT_APP_API` - `http://localhost`
: Here you can set the URL for the middleware. I'm not sure why it doesn't
contain a port.
; `MIDDLEWARE_IMAGE` - `registry.deploy.opusvl.net/careprotect/c19app-middleware`
: Changes the middleware image. Use this to avoid overwriting your local copy of
the registry image.
; `MIDDLEWARE_IMAGE_VERSION` - `latest`
: Use this if we ever upload a version besides latest, and you want to use it.
; `FRONTEND_HOSTNAME` - `localhost`
: The middleware app needs to know the URL that the frontend will be posting
from, so it can correctly set security cookies and stuff.
; `PORTBASE` - 383
: The "base" port for the EHRBase image to run on. The real port will be this
and then `82` on the end.
; `SERIAL` - `S00382`
: Opus-specific value that should not be a separate variable, but is.
; `POSTGRES_USER` - `postgres`
: Postgres user for EHRDB image
; `EHRBASE_USER` - `ehrbase`
: Postgres user for EHRBase itself, I guess. As in the user EHRBase will use to
connect.

## Deploy Script

The deploy script builds the `nginx.conf` from `.env` via a template, copies the configuration to your Nginx server and reloads the Nginx service.

If you plan to run plain http you will need to remove the certificate references and change the server setup in `nginx.template.conf`, or build your own `nginx.conf` and skip the use of `./deploy`.

You will need to have python, jinja2 and python-dotenv to template the `nginx.conf`

```shell
sudo apt install pip
sudp pip install jinja2 python-dotenv
```

### Certificates

The `nginx.conf` is built to provide access via three url subdomains, `app`, `api` and `ehrbase`, eg.

https://app.subdomain.domain.tld

Certificates are obtained for all the URLS using certbot and placed in one certificate file using the command:

```shell
sudo certbot certonly -d subdomain.domain.tld -d app.subdomain.domain.tld -d api.subdomain.domain.tld -d ehrbase.subdomain.domain.tld
```

## EHRBase

When up and running and API documented here: [http://localhost:38382/ehrbase/swagger-ui.html](http://localhost:38382/ehrbase/swagger-ui.html)

I have added it to port `${PORTBASE}82` in order that it can be accessed for template uploads. It is protected by the passwords and a IP filter at the Nginx reverse proxy. If you are not from a `TRUSTED_IPS` (see below) then you cannot access the service.

## EHRDB

Database not required to listen on external ports. 

PostgreSQL data folder located at `${CONTAINER_VOLUME}/${SERIAL}/postgres` or `/srv/container/volumes/PROJECT/postgres` based on `.env` settings

## Front End

Available at [http://localhost:38380](http://localhost:38380)

### Environment Variables

To get these into a static node.js environment you need to inject them into a javascript file at runtime. I based the `Dockerfile` and `nginx-entrypoint.sh` script on the work shown here: [https://medium.com/@jans.tuomi/how-to-use-environment-variables-in-a-built-frontend-application-in-an-nginx-container-c7a90c011ec2](https://medium.com/@jans.tuomi/how-to-use-environment-variables-in-a-built-frontend-application-in-an-nginx-container-c7a90c011ec2)

I chose not to include the gettext package and used a simple `render_template` function I use in bash for this kind of thing.

TRUSTED_IPS are set in the `nginx.conf` and as an environment variable should look like this:

```sehll
TRUSTED_IPS="10.0.0.0/8 yes; 172.16.0.0/12 yes; 192.168.0.0/16 yes;"
```

That goes into the template like this:

```jinja2
geo $admin_{{ env['SERIAL'] }} {
    default no;
    {{ env['TRUSTED_IPS'] or '10.0.0.0/8 yes; 172.16.0.0/12 yes; 192.168.0.0/16 yes;' }}
}
```

## Middleware

Available at [http://localhost:38381](http://localhost:38381)
