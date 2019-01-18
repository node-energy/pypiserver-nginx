## Setup

* Deploy a droplet on Digital Ocean having Docker preinstalled (one click apps). The cheapest one will do the job.
* ssh into it
* Clone this repository and cd into it
* `docker-compose up` and it's running

### Create `.htpasswd` file

First, you need to create an `.htpasswd` file:

`htpasswd -c config/nginx/auth/.htpasswd username`

If you want to add more users, call

`htpasswd onfig/nginx/auth/.htpasswd second_username`

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
