# Getting started for the impatient

Goals of this document:

-   Getting started guide
-   Short overview of Platform.sh
-   Define the needed tools
-   Local setup (note the major versions of PHP and MariaDB)
-   Explain the ideal workflow with Platform.sh
-   Explain access rights
-   Drupal specific steps
    -   distribution
    -   Drush make
    -   vanilla
    -   migrate existing project
    -   custom code (modul, external repo - drush make)

What is Platform.sh? Platform.sh is a groundbreaking hosting and
development tool for web applications. It extends a branch-merge Git
workflow to infrastructure so that every branch has its own URL and can
be tested as if it were in production. Platform.sh architecture allows
it to scale for the largest sites.

Note: The environments used in Platform.sh are deployed on a read-only
filesystems and are rebuilt with every Git push.

# Drupal

## How to set up your local Drupal development

This guide will teach you how to set up local Drupal development for use
with Platform.sh.

### Prerequisites

To succeed with Platform.sh you need the following installed on your
local machine:

#### For the Platform.sh CLI:

-   [id_rsa public private keypair](https://help.github.com/articles/generating-ssh-keys/)
-   [Git](http://git-scm.com/)
-   [Composer](https://getcomposer.org/)
-   [The Platform.sh CLI](https://github.com/platformsh/platformsh-cli)
-   [Drush](https://github.com/drush-ops/drush)

#### For your Drupal stack:

-   [Nginx](http://nginx.org/) (or Apache) web server
-   [MariaDB](https://mariadb.org/) (or MySQL) database
-   Optional: [Solr](https://lucene.apache.org/solr/),
    [Redis](http://redis.io/)

You will also need to have signed up for a
[Platform.sh](https://platform.sh) project.

Platform.sh is currently running PHP 5.4, MySQL Ver 15.1 Distrib
5.5.40-MariaDB

### Goals

1.  Authenticate locally using the [Platform.sh CLI](https://github.com/platformsh/platformsh-cli)
2.  Upload your SSH public key
3.  Use the [Platform.sh CLI](https://github.com/platformsh/platformsh-cli) to obtain and
    build your project’s repository
4.  Understand Settings.php
5.  Understand Build Modes
6.  Connect to your local database
7.  Use your Drush aliases
8.  Synchronize databases and files with Platform.sh

### Authenticate locally using the Platform.sh CLI

The [Platform.sh CLI](https://github.com/platformsh/platformsh-cli) will
authenticate you with Platform.sh and show your projects. Just type this
command to start:

```bash
$ platform
```

The credentials you enter are the same as your [Platform.sh account](https://accounts.platform.sh/user).

### Upload your SSH public key

You need an [id_rsa public/private
keypair](https://help.github.com/articles/generating-ssh-keys/) to use
Platform.sh.

#### Upload using the Web Interface

To upload the public key in the browser go to [your Platform.sh
account](https://accounts.platform.sh/user) and click the
SSH Keys tab. Name your key in the *Title* field, and paste the public
key into the *Key* field. Your key will typically be found at
`~/.ssh/id_rsa.pub` on Linux and Mac OS X machines.

![Screenshot of a public key field](/images/edit-ssh.png)

#### Upload using Platform.sh CLI

Alternately, you can upload your SSH key using the Platform.sh CLI
itself.

```bash
$ platform ssh-key:add ~/.ssh/id_rsa.pub
```

### Use the Platform.sh CLI to obtain and build your project’s repository

The `platform` command will show you a list of your projects.

```bash
$ platform
Welcome to Platform.sh!
Your projects are:
+---------------+-------------------------------------+-------------------------------------------------+
| ID            | Name                                | URL                                             |
+---------------+-------------------------------------+-------------------------------------------------+
| [PROJECT-ID] | My Drupal Site                      | https://[REGION].platform.sh//#/projects/[PROJECT-ID] |
| [PROJECT-ID] | A Symfony Project                   | https://[REGION].platform.sh/#/projects/[PROJECT-ID] |
```

You can obtain a local copy of the project using the `platform get`
command:

```bash
$ platform get [PROJECT-ID]
```

Now you can see the local directory structure that the Platform CLI
provides for your local development:

```bash
$ ls -1
builds     # Contains all builds of your projects
repository # Checkout of the Git repository
shared     # Your files directory, and your settings.local.php file
www        # A symlink that always references the latest build.
           # This should be the document root for your local web server
```

The `builds` directory contains every build of your project. This is
relevant when you use Drush Make files to assist in your site building.

The `repository` directory is your local checkout of the Platform.sh Git
repository. This is where you edit code and issue normal Git commands,
like `git pull`, `git add`, `git commit`, and `git push`.

The `shared` directory is for your settings.local.php file which stores
the connection details to your local database.

See the section below about Settings.php for a full explanation of the
settings.local.php file.

The `www` symlink is created by the `platform build` command and will
always reference the latest build in the builds directory. The `www`
directory should become your DOCROOT for local development.

### Understand Settings.php

Drupal sites use a file called settings.php to store database connection
details and other important configurations. Platform.sh has a specific
concept for managing settings.php which is important to understand to
succeed. For both the local copy of your site, as well as on the server,
settings.php should be found at sites/default/settings.php, and should
be generated by Platform.sh. That means you will not be committing a
settings.php file to your Git repository in normal circumstances. Here
is the entire contents of a generated settings.php:

```php
 <?php $update_free_access = FALSE;

 $drupal_hash_salt = '5vNH-JwuKOSlgzbJCL3FbXvNQNfd8Bz26SiadpFx6gE';

 $local_settings = dirname(__FILE__) . '/settings.local.php'; if
 (file_exists(\$local_settings)) { require_once(\$local_settings);
 }
```
The important part to see, starting in line 6, is the inclusion of
another file, `settings.local.php`, which will handle the actual
connection to the database, as well as the parsing of other important
environmental variables from Platform.sh.

### Understand Build Modes

Platform.sh offers three build modes for Drupal projects: Vanilla, Drush
Make, and Install Profiles.

> **note**
> You can change build modes by changing the files in your repository. Platform.sh recognizes each mode based on the presence or absence of `project.make` or `*.profile` files.

#### Vanilla build mode

In *Vanilla mode* you simply commit all of Drupal's files directly into
the Git repository instead of using Drush Make.

In this mode, take care not to commit any database credentials to your
repository. The following lines should be present in your repository's
`.gitignore` file, which will guarantee that a settings.local.php file
won't get committed to Git:

``` {.sourceCode .text}
# Ignore configuration files that may contain sensitive information.
sites/*/settings*.php
```

#### Drush Make build mode

Drush Make build mode looks for a `project.make` file which will get
executed during the build process.

The default `project.make` file for a Drupal 7 installation looks like
this:

```ini
api = 2
core = 7.x

; Drupal core.
projects[drupal][type] = core
projects[drupal][version] = 7.38
projects[drupal][patch][] = "https://drupal.org/files/issues/install-redirect-on-empty-database-728702-36.patch"

; Platform indicator module.
projects[platform][version] = 1.3
projects[platform][subdir] = contrib
```

#### Install Profile build mode

If your project contains a profile file: `*.profile`, the Platform.sh
CLI builds your project in profile mode. This is similar to what
Drupal.org does to build distributions. Everything you have in your
repository will be copied to your `profiles/[name]` folder.

> **note**
> It is a mistake to mix Vanilla mode with other modes. If you've copied all of the Drupal core files into your repository then you need to make sure you don't have any `*.make` or `*.profile` files.

#### Database credentials

You need to add your local database credentials to a
`settings.local.php` file.

This will be stored in `sites/default/settings.local.php` in your local
Drupal site. However, if you have used the CLI to build your project,
then it's better to use `shared/settings.local.php` (inside the project
root). The CLI will have created this file for you, when you ran the
`platform get` or `platform build` command.

> **note**
> If you are using the CLI but there is no `shared/settings.local.php` file, re-run `platform build`.

```php
<?php
// Database configuration.
$databases['default']['default'] = array(
  'driver' => 'mysql',
  'host' => 'localhost',
  'username' => '',
  'password' => '',
  'database' => '',
  'prefix' => '',
);
```

> **note**
> You never have to add the server-side database credentials to `settings.local.php`. Platform.sh generates a `settings.php` for each environment, already containing the proper database credentials.

### Drush Aliases

The [Platform.sh CLI](https://github.com/platformsh/platformsh-cli)
generates and maintains Drush Aliases that allow you to issue remote
Drush commands on any environment (branch) that is running on
Platform.sh. There is also a Drush Alias for your local site.

To see your Drush Aliases, use the `platform drush-aliases` command:

```bash
$ platform drush-aliases
Aliases for My Site (tqmd2kvitnoly):
    @my-site._local
    @my-site.master
    @my-site.staging
    @my-site.sprint1
```

> **note**
> Run local Drush commands with `drush`. Run remote Drush commands with `platform drush`. Any `platform drush` command will execute on the remote environment that you currently have checked out.

### Synchronize Databases and Files with the Platform CLI

Given the Drush aliases shown above, you can now use a normal Drush
command to synchronize my local database with the data from my Master
environment online:

```bash
$ drush sql-sync @my-site.master @my-site._local
```

In the same style, use Drush to grab the uploaded files from the files
directory and pull them into your local environment:

```bash
$ drush rsync @my-site.staging:%files @my-site._local:%files
```

> **note**
> Never commit the files that are in your `files` directory to the Git repository. Git is only meant for code, not *data*, and files that are managed by your Drupal site are considered data.

#### SQL-sync troubleshootings

Drush 7 has problems with SQL-syncing Drupal 7 sites. If you see error
below:

```bash
Starting to dump database on Source. [ok]
Directory /app exists, but is not writable. Please check directory permissions. [error]
Unable to create backup directory /app/drush-backups/main. [error]
Database dump saved to /tmp/main_20150206_091052.sql.gz [success]
sql-dump failed.
```

Than you should downgrade to Drush version 6.\* to make sql-sync works:

```bash
$ composer global require 'drush/drush:6.*'
```

### IDE Specific Tips

MAMP pro:

In order for MAMP to work well with the symlinks created by the
[Platform.sh CLI](https://github.com/platformsh/platformsh-cli), add the
following to the section under Hosts \> Advanced called “Customized
virtual host general settings.” For more details visit [MAMP Pro
documentation
page](http://documentation.mamp.info/en/documentation/mamp/).

```bash
<Directory />
        Options FollowSymLinks
        AllowOverride All
</Directory>
```

> [Laravel Forum
> Archives](http://forumsarchive.laravel.io/viewtopic.php?pid=11232#p11232)

> **note**
> When you specify your document root, MAMP will follow the symlink and substitute the actual build folder path. This means that when you rebuild your project locally, you will need to repoint the docroot to the symlink again so it will refresh the build path.
