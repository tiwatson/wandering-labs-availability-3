# How to provision a new environment for wandering-labs-availability-3

This project uses the open source [capistrano-mb](https://github.com/mattbrictson/capistrano-mb) set of tasks for capistrano to provision new environments. This document contains a quick overview of the provisioning process.

In capistrano terminology, *stage* refers to an environment like `staging` or `production`. These instructions will use the *stage* term for the remainder of the document.

In this document, *provisioning* refers to preparing a server for deployment, including the following steps:

* Installing Nginx, Redis, and PostgreSQL
* Compiling Ruby
* Creating the database
* Setting passwords and other configuration


## Prerequisites

Capistrano runs on your local machine and uses SSH to perform the deployment on the remote server. Therefore:

* The Capistrano gem must be installed (see `README.md` for project setup instructions).
* You must have SSH access to the server.
* Your SSH key must be installed on the server.
* Your account on the server must have `sudo` access.
* Your account must be able to run `sudo` without be prompted for a password. See [How to run sudo command with no password?](http://askubuntu.com/questions/192050/how-to-run-sudo-command-with-no-password).

Furthermore, the server itself must meet the following requirements:

* Ubuntu 14.04 LTS (64 bit).


## 1. Create a new stage (or edit an existing one)

Stages are defined as `.rb` files in the `config/deploy/` directory. The name of the file becomes the name of the stage when executing capistrano commands. For example, the production stage is defined in `config/deploy/production.rb`.

Create a new stage (or modify an existing one to move that stage to a new server address, for example) using the existing stage files as examples. The stage file describes the IP address of the server and other stage-specific information.

## 2. Run the provision command

`bundle exec cap <stage> provision:14_04`

Replace `<stage>` with the name of the stage you wish to provision (e.g. `production`).

You will be prompted to choose passwords and enter other configuration values as the script runs.

It is safe to run the provision command multiple times on the same stage (the scripts are generally idempotent).

## 3. Install a custom SSL key and certificate using Let's Encrypt with auto-renewal

Perform the following commands as root.

```
mkdir -p /opt/certbot
cd /opt/certbot/
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
./certbot-auto certonly
```

When prompted:

* Select the *webroot* option
* Enter domains to secure (for example): `reserve.wanderinglabs.com`
* Enter webroot path: `/home/deployer/apps/wandering_labs_availability_3/shared/public/`

The `.pem` files created by `certbot-auto` need to be linked so Nginx can see them:

```
ln -s /etc/letsencrypt/live/reserve.wanderinglabs.com/fullchain.pem /etc/ssl/wandering_labs_availability_3.crt
ln -s /etc/letsencrypt/live/reserve.wanderinglabs.com/privkey.pem /etc/ssl/wandering_labs_availability_3.key
```

Restart Nginx to enable the certificate:

```
service nginx restart
```

Verify auto-renewal works by doing a dry run:

```
/opt/certbot/certbot-auto renew --webroot-path /home/deployer/apps/wandering_labs_availability_3/shared/public/ --dry-run
```

Finally, add this to the root crontab to enable auto-renewal:

```
54 4,16 * * * /opt/certbot/certbot-auto renew --quiet --no-self-upgrade --webroot-path /home/deployer/apps/wandering_labs_availability_3/shared/public/
```

## 4. Deploy

Once provisioning is complete, deploy the application using the capistrano instructions in `DEPLOYMENT.md`.
