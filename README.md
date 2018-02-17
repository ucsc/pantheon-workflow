# A Development workflow for Pantheon.io

_note: This article assumes the reader is familiar with `Git` and has a local development environment such as [XAMPP](https://www.apachefriends.org/index.html) installed on their computer_

## Introduction

[Pantheon.io](https://pantheon.io) is a robust [WordPress](https://wordpress.org) and [Drupal](https://drupal.org) hosting service that provides three online environments per site: `DEV`, `TEST` and `LIVE`. Pantheon enforces a workflow whereby one "develops" on `DEV`, "tests" on `TEST` and you point your domain name to `LIVE`. In order to update a site, one updates `DEV` (WP update, plugin update, etc.). In order to test the update, Pantheon pushes the new code from `DEV` and pulls the database from `LIVE` into `TEST`. If all tests well, the new code is then pushed to `LIVE`.

Pantheon also maintains `Git` repos of each of these environments, which a developer may pull from and push to when developing in an local development environment such as WAMPP\MAMPP\LAMPP. (While all three environments are available via a `Git` repo, it is _highly_ recommended that a developer only utilize the `DEV` repo.)

![Image](https://s3-us-west-1.amazonaws.com/mollusk/UCSC/pantheon-dev-workflow.png "Pantheon Development Paradigm")

From a local development standpoint, a significant drawback of this setup is that the Pantheon repositories are comprised of the _entire_ WordPress (or Drupal) install. (_Almost_ the entire install -- Pantheon excludes the uploads directory, eg. `wp-content/uploads/` in WordPress -- more on that below.)

The reason that this is a drawback is because this configuration does not allow a developer to develop a theme or plugin in a separate repository located someplace else, such as [Github](https://github.com). Pantheon does not permit the creation or cloning of additional repositories _within_ the larger repository of one's development site.

The methodology described in this document was developed as a way to overcome this drawback.

## Requirements

The following are either requirements or strongly recommended:

- Local development server such as [XAMPP](https://www.apachefriends.org/index.html) and the ability to host multiple installs via `<VirtualHosts>`
- [Git](https://Git-scm.com/)
- [WP-Cli](http://wp-cli.org/) (Recommended)
- [Terminus](https://pantheon.io/docs/terminus/install/) (Recommended)
- [wp-sync-db](https://github.com/wp-sync-db/wp-sync-db)

## The workaround

_note:_ _as primarily a WordPress developer, the following examples are based on WordPress; however, the same principles apply to Drupal development_

The workaround for Pantheon's strict `Git` repo policy is to create _two_ sites on your local development machine. One site is simply a clone of your Pantheon `DEV` site, with its strict Git policy.

The second local install is for developing your theme or plugins, which can be committed, pushed to and pulled from their own respective `Git` repos. 

It is important that you have two **complete** WP installs -- WordPress Core, database, etc. -- as this setup requires keeping both databases and `wp-content/uploads/` drectories in sync.

## The constituent parts

### The scripts

The scripts in this repository are [bash](https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php) scripts.

The rsync commands for the `pull-content` and `push-content` scripts were written with the help of Pantheon's [rsync and SFTP](https://pantheon.io/docs/rsync-and-sftp/) documentation.

The following describes the scripts in this repository:

- **pull-content:** This script uses [rsync](https://rsync.samba.org/) to sync the `wp-content/uploads/` directories from your Pantheon `DEV` install to your local environment. As mentioned above, for Git efficiency Pantheon omits this directory and the subdirectories contained therein from the repo. This script may be used in both local development environments, as each environment will need this data locally in order for the _work_ to _flow_ properly.
- **push-content:** This script uses [rsync](https://rsync.samba.org/) to sync the `wp-content/uploads/` directory of your local development environment with the Pantheon `DEV` environment. This script is not used as frequently as `pull-content`, as most of the time one is adding content and files directly to the Pantheon `DEV` environment (this especially true if many people are working on content development for a site). But if you begin your project locally and want to push your initial work up to Pantheon, this is the script to do it with.
- **synctheme:** This script will sync the files of your theme or plugin from your local install containing clones of those repos to the local install of the Pantheon repo.

- **rsync-exclude.txt:** this file is similar to a `.gitignore` file but for rsync. It lists all the files and directories you want to exclude when you run the `synctheme` script. At the very least it should include your `.git/` directory and your `.gitignore` file.

### The plugin

As mentioned above, this workflow depends on two _complete_ WordPress installs on your local development machine, including MySQL databases. While the scripts in this repository will keep the files in sync, we will use a special WordPress Developer's plugin called [wp-sync-db](https://github.com/wp-sync-db/wp-sync-db) to keep the databases in sync. This plugin will need to be installed and activated on both local WordPress installs.

- **[wp-sync-db](https://github.com/wp-sync-db/wp-sync-db):** A WordPress developer's plugin that enables syncing WordPress databases between installs. The link is to a github repo. The plugin is free.

(The same concept applies to Drupal; however, I am unaware of a similar Drupal developer's module that will achieve the same thing.)

## Editing the scripts

_note: As described above, these are [bash](https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php) scripts comprised of [rsync](https://rsync.samba.org/) commands. They can be edited in any text editor, just be sure to maintain the [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) located at the top of the file and make them executable._

There are a few edits you need to make to make these scripts work for your project.

### Pull Content

Contents of the `pull-content` script.

```shell
#!/bin/bash

rsync -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b@appserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b.drush.in:code/wp-content/uploads/. ~/public_html/wptest/wp-content/uploads/.

```

This string, `81cf9d89-c08b-419a-a74c-ffcdcd84766b` is what identifies your particular Pantheon site. This string, `dev` identifies the Pantheon environment you're working in , `dev`, `test`, or `live` (again, it is highly recommended you stick with the `dev` repo when developing a new site). This information is found in the dashboard of your Pantheon site at the upper right, **Connection Info** (see image below). Replace the information from your `dev` site with the one in this git repo.

This string, `~/public_html/wptest/wp-content/uploads/.`, is the path to your local installations' respective `/wp-content/uploads/` folders (be sure to include the `.` at the end of your path).

### Push Content

Contents of the `push-content` script.

```shell
#!/bin/bash

rsync -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' ~/public_html/wptest/wp-content/uploads/. --temp-dir=~/tmp/ dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b@appserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b.drush.in:files/

```

This script is the reverse of the one above it. Replace the same information as above from your Pantheon site and your local `wp-content/uploads/` in order for this to work.

### Synctheme

Contents of the `synctheme` script.

```shell
#!/bin/bash

rsync -rvz --progress --exclude-from=/home/jason/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/rsync-exclude.txt ~/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/ ~/public_html/ucsc-communications/wp-content/themes/ucsc-comm-genesis-child/

```

This script goes inside your local _non-pantheon_ install. A copy of this script should reside inside the `root` directory of each piece of custom functionality you're developing. 

- If you are developing a custom theme, this script goes inside `wp-content/themes/name-of-my-theme/`. 
- If you are developing a custom plugin, this script goes inside `wp-content/plugins/name-of-my-plugin/`.
- (ie., whichever directory you would normally perform `git add .`, `git commit -m "my message"`, and `git push origin master`.)

This script will sync files _from_ the development directory of your theme or plugin _to_ a directory of the same name inside your local Pantheon install (_note: the directory names must be the same for each local install_).

### rsync-exclude.txt

From the `synctheme` script:

`--exclude-from=/home/jason/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/rsync-exclude.txt`

Like a `.gitignore` file, this file lists all the files and directories to _exclude_ when you sync your theme or plugin with your local Pantheon Dev site. At the very least, it should list your `.git/` directory, `.gitignore` file and the names of your other scripts. If you develop using a package manager such as [npm](https://www.npmjs.com/), you might also include any non-essential development directories it installs, such as `node_modules/`. Here is the current contents of the `rsync-exclude.txt` file in this repository:

~~~shell
.git/
.gitignore
.vscode/
node_modules/
synctheme
pull-content
push-content
rsync-exclude.txt
~~~

## Script notes

### Absolute paths

The paths in these scripts are *absolute*, or full paths (eg., `/home/jason/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/rsync-exclude.txt`). The reason for this is so that each script can be placed _anywhere_ on your machine to run. As described above, the `synctheme` script resides in my theme's `root` directory, ie., the directory I'm pulling from and pushing to in Git. Using absolute paths, I can run the script from within my working directory and be sure that the proper directory structure is used.

### Rsyc flags / Do a dry-run

Like many [Unix](https://en.wikipedia.org/wiki/Unix)-like programs, you can pass options to your command using "flags." Flags are notated using either single or double dashes `-v` or `--verbose` will both run the script with verbose output. The `pull-content` and `push-content` rsync commands each use the following flags (as described in the [Pantheon rsync and SFTP](https://pantheon.io/docs/rsync-and-sftp/) documentation): `-rlvz`, `--size-only`, `--size-only`, `--ipv4`, `--progress`, and `-e`. Before running a script from within your particular development environment, it is helpful to do a __dry run__ of your script by adding the following additional flag:

`--dry-run`

This flag can be added anywhere within the sequence of the other tags. I often just put it first:

`rsync --dry-run -rlvz --etc --etc --etc`

Once I run the script (or simply paste the entire rsync command into my terminal) with the `--dry-run` flag and make sure it's doing what I want it to do, I'll remove that flag for future script runs.

### Make scripts executable

In addition to editing the command inside the script, you may also rename your scripts to any name that makes sense to you. If the script is not executable after you clone this repo, or if it _becomes_ not executable due to your edits, you can make it executable by issuing the following command in your terminal:

~~~shell
user@computer:~$ chmod +x scriptName
~~~

### Executing a script

Once you get your rsync commands edited properly, it's time to run it! Here is how:

~~~shell
user@computer:~/public_html/wptest/wp-content/themes/my-custom-thenme$./synctheme
~~~

... which is to say `./synctheme`

#### Pantheon Environment Connection Info

![Image](https://s3-us-west-1.amazonaws.com/mollusk/UCSC/pantheon-dashboard-connection-info.png "Image Title")

## Set up your local environments

As described above, this workflow requires _two_ local installations. Setting up a local web development environment is beyond the purview of this article. It is also assumed that you have [generated and added your SSH Key to Pantheon](https://pantheon.io/docs/ssh-keys/). 

- __Pantheon `dev` clone__
- __Custom Theme and Plugin install__

### Pantheon `dev` clone

#### Database

Create a new _empty_ MySQL database in your local phpMyAdmin either directly in the web UI or via the command line. Give your user permissions to access it.

You may download a copy of your `dev` site's database from its dashboard and upload it to your newly created local database; however, for this setup, I recommend using [wp-sync-db](https://github.com/wp-sync-db/wp-sync-db), the developer's plugin listed at the top, to pull the dev database down to your local install.

For this reason, for the time being, we are going to leave it empty. Once the site is cloned and launched locally, it will run the WordPress installation script and you will get the standard "Hello World" database content. 

#### Clone Pantheon dev

You will find your `dev` site's clone command in its dashboard by clicking the big orange __Clone with Git__ button at the top right and copying the command. The __Development Mode__ of your site must be set to __Git__.

![Image](https://s3-us-west-1.amazonaws.com/mollusk/UCSC/pantheon-clone.png "Clone from Pantheon")

Enter your [Apache](https://httpd.apache.org/) or [Nginx](https://www.nginx.com/) document root and paste the command you copied from your Pantheon `dev` dashboard into your terminal:

~~~shell
user@computer:~/public_html/git clone ssh://codeserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b@codeserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b.drush.in:2222/~/repository.git ucsc-communications
~~~

#### wp-config-local.php

In order to develop a Pantheon site locally, a local configuration file, `wp-config-local.php` is needed along with the standard `wp-config.php` file, which is actually not so standard, as it's customized for Pantheon's servers. WordPress installs also come with a `wp-config-sample.php` file. The best way to set up your local config is to make a copy of `wp-config-sample.php` and rename it to `wp-config-local.php`.

~~~shell
user@computer:~/public_html/my-pantheon-dev-site/$ cp wp-config-sample.php wp-config-local.php
~~~

Enter the database name, user and password in the appropriate places in yonur new `wp-config-local.php` file.

#### Fire up your new local Pantheon dev site

Now that you have the database set up and the Pantheon dev site cloned, it's time to visit the local URL you are using to identify it with in your `<VirtualHosts>` file (eg: http://my-pantheon-site.local). If all goes well, you should see the WordPress install script:

![Image](https://s3-us-west-1.amazonaws.com/mollusk/UCSC/wordpress-install.png "Clone from Pantheon")