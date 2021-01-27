# EHRBase with Middleware and C19 front end

In order to pull from the harbour registry you will need to use `docker login registry.deploy.opusvl.net`

## Deploy Script

The deploy script build the `nginx.conf` from `.env` via a template, copies the configuration to your Nginx server and reloads the Nginx service.

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

## Environment Variables

Copy the `.env.example` file to `.env` and modify as necessary.

```shell
FRONTEND_HOSTNAME=app.c19.staging3.opusvl.com

REACT_APP_API=https://api.c19.staging3.opusvl.com/c19-alpha/0.0.1
```

These are important and help the middleware and frontends resolve CORS issues.

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