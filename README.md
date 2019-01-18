## Todo
* less duplication of url in configuration
* Configure django-timeseries' CircleCI to automatically build and upload.
* Resolve why packages are not stored in packages folder

## Get the server to run

### Initial setup

* Deploy a droplet on Digital Ocean having Docker preinstalled (one click apps). The cheapest one will do the job.
* Set your DNS records up, so you have a URL for that IP (required for Let's encrypt)
* ssh into it
* Clone this repository and cd into it
* Install `htpasswd` to setup http authentication: `apt install apache2-utils`

### Create `.htpasswd` files

Next, you need to create a `.htpasswd` file for downloading permissions:

`htpasswd -c config/nginx/auth/.htpasswd username`

If you want to add more users, call

`htpasswd config/nginx/auth/.htpasswd second_username`

Create another `.htpasswd` file for uploading permissions:
`htpasswd -c data/.htpasswd upload_username`

Users allowed to upload packages must also be listed in the nginx .htpasswd

`htpasswd config/nginx/auth/.htpasswd upload_username` # enter same password as before

### Obtain SSL certificate
Now it's time to get a SSL certificate. There is a chicken and egg problem involved here.
You need to have a certificate for nginx to start but you need nginx to be running to serve the challenge required by [Let's Encrypt](https://letsencrypt.org/).
Luckily [wmnnd](https://github.com/wmnnd/nginx-certbot) wrote a script that solves this issue.
It's part of this repository, so you can simply run `./init-letsencrypt.sh`.

### Start
Finally you can start the server. It's as simple as `docker-compose up`.


### Auto start on system start
You don't want to ssh into the server every time the server did a reboot. 
The easiest way to ensure it auto starts on system boot is setting up a systemd service.
This repository contains a configuration for that service.
You can enable it by: 

```
cp docker-compose-privatepypi.service /etc/systemd/system/
systemctl enable docker-compose-privatepypi
```

## Usage

### Build

`python setup.py sdist bdist_wheel`

### Upload
Somehow reading from environment variables didn't work for me, at least not locally.

`twine upload dist/* --repository-url $TWINE_REPOSITORY_URL -u $TWINE_USERNAME -p $TWINE_PASSWORD`

### Naming
By default the private PyPI server will fallback to the https//pypi.org/ if the desired package is not found.
Therefore you'll want to name your packages in a way they don't clash with existing packages on PyPI.
Otherwise this can become a source of hard to track bugs because a possibly failing install won't fail loudly.


## Install packages from this repository
Configure Pipfile similar to this:
```
[[source]]
name = "node-energy"
verify_ssl = true
url = "https://user:pass@pypi.node.energy"
```

`pipenv install package-name --index node-energy`

Alternatively, you can install it without configuring the index in `Pipfile`: 

`pipenv install package-name --index https://user:pass@pypi.node.energy/`

Unfortunately there does not seem to be a way to configure the access via environment variables using pipenv at the moment.
This is the reason why you should set up separate users with upload permissions. 
For these, you can safely store the credentials in your CI environment variables.
So you only have to check in the reading credentials into your codebase.
Let's hope `pipenv` will support a better way to pass in credentials in the future.

## Credits:
Thanks for inspiration:
* https://pawamoy.github.io/2018/02/01/docker-compose-django-postgres-nginx.html
* https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71
* https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up
