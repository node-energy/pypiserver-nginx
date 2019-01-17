### Create `.htpasswd` file

First, you need to create an `.htpasswd` file:

`htpasswd -c config/nginx/auth/.htpasswd username`

If you want to add more users, call

`htpasswd onfig/nginx/auth/.htpasswd second_username`

