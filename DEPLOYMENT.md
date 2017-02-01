# How to deploy wandering-labs-availability-3

This document covers the steps need to deploy the application to an existing environment. To create a new environment, refer to `PROVISIONING.md`.

* **Developers:** this project is built and deployed using Capistrano, which is the `cap` command. Refer to the *Capistrano* section.
* **System administrators:** once deployed, the various processes and configuration can be maintained with familiar Linux commands. Refer to the *Server maintenance* section.


## Capistrano

### Environments

* **staging** is deployed from the `development` branch.
* **production** is deployed from the `master` branch.

### Prerequisites

Capistrano runs on your local machine and uses SSH to perform the deployment on the remote server. Therefore:

* The Capistrano gem must be installed (see `README.md` for project setup instructions).
* You must have SSH access to the production/staging server.
* Your SSH key must be installed on the server in `~deployer/.ssh/authorized_keys`.
* You must have SSH access to Git repository (using your SSH key).

### Performing a graceful deployment (no migrations)

This will deploy the latest code from `development`. Make sure to `git push` your changes first, or they will not apply.

```
bundle exec cap production deploy
```

### Performing a full deployment

If there are data migrations or other changes that require downtime, perform the deployment using the following task:

```
bundle exec cap production deploy:migrate_and_restart
```

This will stop the app and display a maintenance page during the deployment.


## Server maintenance

This application consists of two executables that run as the `deployer` user:

* Unicorn is the application server that services Rails HTTP requests
* Sidekiq is the background worker that services the job queue stored in Redis

These in turn rely on the following services:

* PostgreSQL
* Redis
* Nginx

### Controlling processes

All processes are set up to start automatically when the server boots, and can be controlled using the standard Ubuntu `service` command:

```
sudo service postgresql <start|stop|restart>
sudo service nginx <start|stop|restart>
sudo service unicorn_wandering_labs_availability_3 <start|stop|restart>
sudo service sidekiq_wandering_labs_availability_3 <start|stop>
```

Note that Unicorn and Sidekiq require access to PostgreSQL and Redis, so those supporting services should be started first.

### Configuration

All configuration values required by the application can be changed here:

```
/home/deployer/apps/wandering_labs_availability_3/shared/.env.production
```

After making changes to this file, be sure to restart the application:

```
sudo service unicorn_wandering_labs_availability_3 restart
sudo service sidekiq_wandering_labs_availability_3 stop
sudo service sidekiq_wandering_labs_availability_3 start
```
