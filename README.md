# Digidem Server Provisioning and Configuration

We use [DigitalOcean](https://www.digitalocean.com) for hosting virtual servers. We chose DO over [AWS EC2](https://aws.amazon.com/ec2/) and other options like [RackSpace](http://rackspace.com) because DO is very easy to use and they donated free credit to our account as a 501c3 non-profit.

We use [dokku](http://dokku.viewdocs.io/dokku/) as a mini [Heroku](https://heroku.com) running on a single server. It allows us to deploy apps as simply as `git push dokku`. We run several apps on the same server since our server load is very low right now.

Digital Ocean have a pre-installed Dokku droplet, but we make some tweaks:

1. We upgrade [nginx](http://nginx.org) to the latest version, which allows us to turn off [proxy_request_buffering](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_request_buffering) which speeds up submitting images with Simple-ODK by streaming the uploaded forms directly to our `simple-odk` node server app.
2. We add an 8GB swap file
3. We lock down ssh and block repeated access attempts
4. We use iptables to block all ports apart from 22, 80, 443
5. We tweak nginx [worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections) to the max supported by the server

For details of these tweaks see [cloud-init.sh](./cloud-init.sh) which is the script used to configure the server.

Currently we maintain two servers: a 1gb production server at `apps.digital-democracy.org` and a 512mb development server at `dev.digital-democracy.org`

## Install

The scripts in this repo are little helpers to provision and configure a server. Clone this repo and change into the directory:

```sh
git clone https://github.com/digidem/digidem-server.git
cd digidem-server
```

## Provision

To provision a new server for Digital Democracy with default options simply run:

```sh
export DO_API_KEY=our_digitalocean_api_key
./provision
```

This will create a new 1gb 'droplet' (server) with dokku installed in `nyc3` region, and configure it with all the tweaks we need. You will need to visit https://cloud.digitalocean.com/ to find out the IP of the newly created droplet for the next step.

To see other available options type `./provision --help`

## Setup

We ideally want a DNS name to point at the new server. We manage our DNS with [CloudFlare](https://www.cloudflare.com). Visit the DNS management page and add an `A` record for the new server IP address.

Visit the new server IP address in the browser and add you should see the Dokku setup page. Update the Hostname to the new address you just added to Cloudflare, and select 'virtual host naming'.

## Configure

If you have provisioned (created) a new server with dokku installed and you just want to run the configure script, you can run `./configure`. DO NOT run this command twice, or on a server that has already been provisioned with the script above, since it does not do any checks on repeat configuration right now and may mess things up.

## Connecting to the server

```sh
ssh root@server_hostname
```

You can then run remote commands on the server, such as `dokku apps:create myapp`

## Adding public keys

If you want additional users to be able to access the server or dokku, you need to add their public keys.

#### Adding a key to the server

Adding a users key to the server will give them root access. Be careful with this. This is not needed for deploying new apps, only for server maintenance.

```sh
cat /path/to/public_key | ssh root@server_hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

#### Adding a key to dokku as an admin user

This will allow a user with the public key to push new apps dokku and other dokku admin functions.

```sh
cat /path/to/public_key | ssh root@server_hostname "sudo sshcommand acl-add dokku [description]"
```

`[description]` can be anything as long as it is one word only (e.g. "personal", "home", etc.).

## Deploying a new or existing app

To deploy a new app you need to add a git remote for dokku:

```sh
cd /path/to/my/app
git remote add dokku dokku@server_hostname:app_name
```

Now you can `git push dokku master` to deploy your app to dokku. See [dokku docs](http://dokku.viewdocs.io/dokku/application-deployment/) for more details.

The easiest way to run dokku commands locally is to install [dokku-toolbelt](https://github.com/digitalsadhu/dokku-toolbelt):

```sh
npm install -g dokku-toolbelt
```

Then when in your app folder (you need to already have the dokku git remote configured) you can run commands like `dt config:set MY_ENV_VARIABLE=some_secret_sauce`

## Optional: Install New Relic server monitoring

We use [New Relic](http://newrelic.com) for basic server monitoring. Connect to the server and follow the New Relic [install instructions](https://docs.newrelic.com/docs/servers/new-relic-servers-linux/installation-configuration/servers-installation-ubuntu-debian)

