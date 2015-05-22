# Wordpress Themes and Plugins

Wordpress themes and plugins for the following publishing services at Malmö stad:
* External Blog
* Intranet Blog
* Intranet News

## Dependencies

* Wordpress >= 4.1
* Wordpress compatible database
* LDAP server for authentication
* Nexus Hybrid Access Gateway for SSO authentication
* [Global Assets](https://github.com/malmostad/global-assets).
* [Avatar service](https://github.com/malmostad/intranet-dashboard/wiki/Avatar-Service-API-v1).
* Ruby for build and deployment with Capistrano

We use [Puppet](https://puppetlabs.com/) in standalone mode to setup server and development environments, see [puppet-mcommons](https://github.com/malmostad/puppet-mcommons/) for in-depth details.


## Development Setup

Development dependencies:

* [Vagrant](https://www.vagrantup.com/)
* A Vagrant compatible virtual machine such as VirtualBox or VMWare

To get the project files and create a Vagrant box with a ready-to-use development environment on your own machine, run the following commands:

```shell
$ git clone git@github.com:malmostad/wp-apps.git
$ cd wp-apps
$ vagrant up <application-name>
```

Where `<application-name>` is one of `internal-news`, `internal-blog` or `external-blog`. If you just run `vagrant up` you will create Vagrant instances for all three applications.

Check the generated `install_info.txt` file in the project root for database details when the provisioning has finished.

Log in to the Vagrant box as the `vagrant` user and start the application in the Vagrant box:

```shell
$ vagrant ssh
$ cd /vagrant
```

Point a browser on your host system to http://127.0.0.1:8000. Editing of the project files on your host system will be reflected when you hit reload in your browser.

When you run the command above for the first time, it creates an Ubuntu 14.04 based Vagrant box with a ready-to-use development environment for the application. This will take some time. Vagrant will launch fast after the first run.

If you get port conflicts in your host system, change `forwarded_port` in the `Vagrantfile` You might also want to edit the value for `vm.hostname` and `puppet.facter` in the same file or do a mapping `localhost` mapping in your hosts `host` file to reflect that value.


## Server Provisioning

The project contains resources for a standalone Puppet (no master) one-time provisioning setup. Do not run or re-run the provisioning on an existing server if you have made manual changes to config files generated by Puppet. It will overwrite.

On a fresh server running a base install of Ubuntu 14.04:

1. Add `app_runner` as a sudo user.
2. Log on to the server as `app_runner` and download the two provisioning files needed:

        $ wget https://raw.githubusercontent.com/malmostad/puppet-mcommons/master/bootstrap.sh
        $ wget https://raw.githubusercontent.com/malmostad/wp-apps/master/puppet/server.pp

3. Run the provisioning:

        $ sudo bash ./bootstrap.sh

When finished, read the generated `install_info.txt` file in `app_runner`'s home directory for database details.

So, what happened?

* Apache and MySQL are configured and installed as services
* An empty Wordpress database is created
* Log rotating and database backup are configured
* Snakeoil SSL certs are generated as placeholders
* Wordpress core is installed
* `wp-config.php` and `.htacess` are generated

The environment should now be ready for application deployment as described below. The user `app_runner` must be used for all build and deployment tasks.


## Build & Deployment

The Ruby based framework [Capistrano 2](https://github.com/capistrano/capistrano/wiki) is used for build and deployment. It uses your *local copy* of this repo, it *does not* check out from the repo.

The `app_runner` user must be used for all deployment tasks (see Server Provisioning above).

Each theme, `internal-news`, `internal-blog` and `external-blog`, is a child theme of the `master` theme.

The `deploy` Capistrano task does the following:

* Compiles asset files from the master and child themes
* Deploys both the master and child theme to the server
* Installs custom plugins to the server as defined as `:custom_plugins` in `config/deploy.rb`
* Installs third-party plugins to the server as defined as `:remote_plugins` in `config/deploy.rb`

The deployment command defines the stage as the theme name and it's stage separated by a dash. Example: to build and deploy the internal news themes to the production server, run the following command in the projects root:

    $ bundle exec cap internal-news-production deploy

Rollback to the previous version:

    $ bundle exec cap internal-news-production deploy

Both themes and plugins are rolled back.

## Update Wordpress core

To update Wordpress core on the server to the version specified in `config/deploy.rb` with `:wordpress_url` (defaults to latest):

    $ bundle exec cap <child-theme-name>-<stage-name> update_wordpress


## Editing Sass files

Each child themes `stylesheets` directory contains theme specific Sass files and are using Sass files from the `master` theme. Sass will listen for changes to files when you edit them with this command:

    $ cd themes
    $ sass --watch --style expanded <child-theme-directory-name>/stylesheets/application.scss

## Licence
Released under AGPL version 3.
