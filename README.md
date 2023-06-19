## Get the server to run

### Initial setup

* Deploy a droplet on Digital Ocean having Docker preinstalled (one click apps). The cheapest one will do the job for us.
* Set your DNS records up, so you have a URL for that IP (required for Let's encrypt)
* ssh into it
* Clone this repository and cd into it
* Install `htpasswd` to setup http authentication: `apt install apache2-utils`
* Create folder to store packages, create `pypiserver` user and group and give them access

```
sudo addgroup --system --gid 9898  pypiserver
sudo adduser --uid 9898 --ingroup pypiserver --system --no-create-home pypiserver
sudo mkdir data/packages
sudo chown -R pypiserver:pypiserver data/packages
sudo chmod g+s data/packages
```

### Create `.htpasswd` files

Next, you need to create a `.htpasswd` file to restrict access to the server:

`htpasswd -c config/nginx/auth/.htpasswd username`

If you want to add more users, call

`htpasswd config/nginx/auth/.htpasswd second_username`

### Obtain SSL certificate
Now it's time to get a SSL certificate. There is a chicken and egg problem involved here.
You need to have a certificate for nginx to start but you need nginx to be running to serve the challenge required by [Let's Encrypt](https://letsencrypt.org/).
Luckily [wmnnd](https://github.com/wmnnd/nginx-certbot) wrote a script that solves this issue.
It's part of this repository, so you can simply run `./init-letsencrypt.sh`.

### Start
Finally you can start the server. It's as simple as `docker compose up -d`.


### Auto start on system start
You don't want to ssh into the server every time the server did a reboot. 
The easiest way to ensure it auto starts on system boot is setting up a systemd service.
This repository contains a configuration for that service.
You can enable it by: 

```
cp docker-compose-privatepypi.service /etc/systemd/system/
systemctl enable docker-compose-privatepypi
```

### Upgrade
To upgrade the dependencies, run `docker compose pull && docker compose down && docker compose up -d`

## Usage

### With poetry

#### Build

Update version number in `pyproject.toml`, then

`poetry build`


#### Upload

Make sure poetry is configured to have access to that PyPI. 

```
poetry config repositories.nodeenergy https://pypi.node.energy/  # no simple!
poetry config http-basic.nodeenergy username password
```

Now you can publish the package

```
poetry publish -r nodeenergy
```

[See here for more information](https://poetry.eustace.io/docs/repositories/#adding-a-repository)

### Other

#### Build

`python setup.py sdist bdist_wheel`

#### Upload
Somehow reading from environment variables didn't work for me, at least not locally.

`twine upload dist/* --repository-url $TWINE_REPOSITORY_URL -u $TWINE_USERNAME -p $TWINE_PASSWORD`

### Naming
By default the private PyPI server will fallback to the https//pypi.org/ if the desired package is not found.
Therefore you'll want to name your packages in a way they don't clash with existing packages on PyPI.
Otherwise this can become a source of hard to track bugs because a possibly failing install won't fail loudly.


## Install packages from this repository

### Via poetry

Add this to the `pyproject.toml`

```
[[tool.poetry.source]]
name = "nodeenergy"
url = "https://pypi.node.energy/simple/"
```

and make sure you configured poetry to have access to the credentials (see section for uploading above).

### Via pipenv
Configure Pipfile similar to this:
```
[[source]]
name = "node-energy"
verify_ssl = true
url = "https://${PRIVATE_PYPI_USERNAME}:${PRIVATE_PYPI_PASSWORD}@pypi.node.energy"
```

and set the user and password environment variables on your system.
Now you can install packages from the private PyPI via:

`pipenv install package-name --index node-energy`

Alternatively, you can install it without configuring the index in `Pipfile`: 

`pipenv install package-name --index https://user:pass@pypi.node.energy/`

This will store your credentials in clear text in your `Pipfile` and `Pipfile.lock`, so you should rather not do this.

### Via pip

`pip install -f https://pypi.node.energy/packages package-name`

This will ask you for your username and password.

## Credits:

Thanks for inspiration:
* https://pawamoy.github.io/2018/02/01/docker-compose-django-postgres-nginx.html
* https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71
* https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up
