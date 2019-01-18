## Get the server to run

### Initial setup

* Deploy a droplet on Digital Ocean having Docker preinstalled (one click apps). The cheapest one will do the job.
* ssh into it
* Clone this repository and cd into it
* Install `htpasswd` to setup http authentication: `apt install apache2-utils`
* `docker-compose up` and it's running

### Create `.htpasswd` file

Next, you need to create an `.htpasswd` file:

`htpasswd -c config/nginx/auth/.htpasswd username`

If you want to add more users, call

`htpasswd onfig/nginx/auth/.htpasswd second_username`

### Obtain SSL certificate
Now it's time to get a SSL certificate. There is a chicken and egg problem involved here.
You need to have a certificate for nginx to start but you need nginx to be running to serve the challenge required by [Let's Encrypt](https://letsencrypt.org/).
Luckily [wmnnd](https://github.com/wmnnd/nginx-certbot) wrote a script that solves this issue.
It's part of this repository, so you can simply run `./init-letsencrypt.sh`.

### Start
Finally you can start the server. It's as simple as `docker-compose up`.

### Setup environment variables
* TWINE_REPOSITORY_URL
* TWINE_USERNAME
* TWINE_PASSWORD

## Usage

### Build

`python setup.py sdist bdist_wheel`

### Upload
Somehow reading from environment variables didn't work for me, at least not locally.

`twine upload dist/* --repository-url $TWINE_REPOSITORY_URL -u $TWINE_USERNAME -p $TWINE_PASSWORD`


## Credits:
Thanks for inspiration:
* https://pawamoy.github.io/2018/02/01/docker-compose-django-postgres-nginx.html
* https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71
