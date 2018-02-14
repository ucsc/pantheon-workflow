# A Development workflow for Pantheon.io

_note: This article assumes the reader is familiar with `Git` and has a local development environment such as [XAMPP](https://www.apachefriends.org/index.html) installed on their computer_

## Requirements

The following are either requirements or strongly recommended:

- Local development server such as [XAMPP](https://www.apachefriends.org/index.html) and the ability to host multiple installs via `<VirtualHosts>`
- [Git](https://Git-scm.com/)
- [WP-Cli](http://wp-cli.org/) (Recommended)
- [Terminus](https://pantheon.io/docs/terminus/install/) (Recommended)
- [wp-sync-db](https://github.com/wp-sync-db/wp-sync-db)

[Pantheon.io](https://pantheon.io) is a robust [WordPress](https://wordpress.org) and [Drupal](https://drupal.org) hosting service that provides three online environments per site: `DEV`, `TEST` and `LIVE`. Pantheon enforces a workflow whereby one "develops" on `DEV`, "tests" on `TEST` and you point your domain name to `LIVE`. In order to update a site, one updates `DEV` (WP update, plugin update, etc.). In order to test the update, Pantheon pushes the new code from `DEV` and pulls the database from `LIVE` into `TEST` in order to test the update. If all tests well, the new code is then pushed to `LIVE`.

Pantheon also maintains `Git` repos of each of these environments, which a developer may pull from and push to when developing in an local development environment such as WAMPP\MAMPP\LAMPP. (While all three environments are available via a `Git` repo, it is _highly_ recommended that a developer only utilize the `DEV` repo.)

From a local development standpoint, a significant drawback of this setup is that the Pantheon repositories are comprised of the _entire_ WordPress (or Drupal) install. (_Almost_ the entire install -- Pantheon excludes the uploads directory, eg. `wp-content/uploads/` in WordPress -- more on that below.)

The reason that this is a drawback is because this configuration does not allow a developer to develop a theme or plugin in a separate repository located someplace else, such as [Github](https://github.com). Pantheon does not permit the creation or cloning of additional repositories _within_ the larger repository of one's development site.

The methodology described in this document was developed as a way to overcome this drawback.

## Two Local development sites, bash scripts, rsync and a text file

_note:_ _as primarily a WordPress developer, the following examples are based on WordPress; however, the same principles apply to Drupal development_

The workaround for Pantheon's strict `Git` repo policy is to create _two_ sites on your local development machine. One site is simply a clone of your Pantheon `DEV` site, with its strict Git policy. The second install is for developing your theme or plugins, which can be committed, pushed to and pulled from their own respective `Git` repos. It is important that you have two **complete** WP installs -- WordPress Core, database, etc. -- The reason for this will be explained later. The scripts included in this repo are used to keep everything in sync.

### Scripts

The scripts in this repository are [Bash](https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php) scripts.

The rsync commands for the `pull-content` and `push-content` scripts were written with the help of Pantheon's [rsync and SFTP](https://pantheon.io/docs/rsync-and-sftp/) documentation.

The following describes the scripts in this repository:

- **pull-content:** This script uses [rsync](https://rsync.samba.org/) to sync the `wp-content/uploads/` directories from your Pantheon `DEV` install to your local environment. As described above, Pantheon omits this directory and the subdirectories contained therein from it's repo (which makes sense, as these can become quite large). This script may be used in both local development environments, as each environment will need this data locally in order to resolve proerly.
- **push-content:** This script uses [rsync](https://rsync.samba.org/) to sync the `wp-content/uploads/` directory of your local development environment with the Pantheon `DEV` environment. This script is not used as frequently as `pull-content`, as most of the time one is adding content and files directly to the Pantheon `DEV` environment. This especially true if many people are working on content development for a site.
- **synctheme:** This script will sync the files of your theme or plugin from your local install containing clones of those repos to the local install of the Pantheon repo.

- **rsync-exclude.txt:** this file is similar to a `.gitignore` file but for rsync. It lists all the files and directories you want to exclude when you run the `synctheme` script. At the very least it should include your `.git/` directory and your `.gitignore` file.

There are a few edits you need to make to make these scripts work for your project.

#### Pull Content

```shell
#!/bin/bash

rsync -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b@appserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b.drush.in:code/wp-content/uploads/. ~/public_html/wptest/wp-content/uploads/.

```

This string, `81cf9d89-c08b-419a-a74c-ffcdcd84766b` is what identifies your particular Pantheon site. This string, `dev` identifies the Pantheon environment you're working in (`dev`, `test`, or `live` -- again, it is highly recommended you stick with the `dev` repo when developing a new site). This information is found in the dashboard of your Pantheon site at the upper right, **Connection Info** (see image below). Replace the information from your `dev` site with the one in this git repo.

This string `~/public_html/wptest/wp-content/uploads/.` is the path to your local installations' respective `/wp-content/uploads/` folders. A copy of this script should go in the root WordPress directories of each of your two local installations, edited accordingly.

#### Push Content

```shell
#!/bin/bash

rsync -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' ~/public_html/wptest/wp-content/uploads/. --temp-dir=~/tmp/ dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b@appserver.dev.81cf9d89-c08b-419a-a74c-ffcdcd84766b.drush.in:files/

```

This script is the reverse of the one above it. Replace the same information as above from your Pantheon site and your local `wp-content/uploads/` in order for this to work.

#### Synctheme

```shell
#!/bin/bash

rsync -rvz --progress --exclude-from=/home/jason/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/rsync-exclude.txt ~/public_html/wptest/wp-content/themes/ucsc-comm-genesis-child/ ~/public_html/ucsc-communications/wp-content/themes/ucsc-comm-genesis-child/

```

##### Pantheon Environment Connection Info

![Image](https://s3-us-west-1.amazonaws.com/mollusk/UCSC/pantheon-dashboard-connection-info.png "Image Title")
