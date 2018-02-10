# A Development workflow for Pantheon.io

_note: This article assumes the reader is familiar with `GIT` and has a local development environment such as [XAMPP](https://www.apachefriends.org/index.html) installed on their computer_

[Pantheon.io](https://pantheon.io) is a robust [WordPress](https://wordpress.org) and [Drupal](https://drupal.org) hosting service that provides three environments per site: `DEV`, `TEST` and `LIVE`. Pantheon enforces a workflow whereby one "develops" on `DEV`, "tests" on `TEST` and you point your domain name to `LIVE`. In order to update a site, one updates `DEV` (WP update, plugin update, etc.). In order to test the update, Pantheon pushes the new code from `DEV` and pulls the database from `LIVE` into `TEST` in order to test the update. If all tests well, the new code is the pushed to `LIVE`. 

Pantheon also maintains `GIT` repos of each of these environments, which a developer may pull from and push to when developing in an local development environment such as WAMPP\MAMPP\LAMPP. (While all three environments are available via a `GIT` repo, it is _highly_ recommended that a developer only utilize the `DEV` repo.)

From a local development standpoint, a significant drawback of this setup is that the Pantheon repositories are comprised of the _entire_ WordPress (or Drupal) install. (_Almost_ the entire install -- Pantheon excludes the uploads directory, eg. `wp-content/uploads/` in WordPress -- more on that below.)

The reason that this is a drawback is because this configuration does not allow a developer to develop a theme or plugin in a separate repository located someplace else, such as [Github](https://github.com). Pantheon does not permit the creation or cloning of additional repositories _within_ the larger repository of one's development site. The methodology described in this document was developed as a way to overcome this drawback.

# Two Local development sites, bash scripts and rsync

_note:_ _as primarily a WordPress developer, the following examples are based on WordPress; however, the same principles apply to Drupal development_

The workaround for Pantheon's strict `GIT` repo policy is to create _two_ sites on your local development machine. One site is simply a clone of your Pantheon `DEV` site, with its strict git policy. The second install is for developing your theme or plugins, which can be committed, pushed to and pulled from their own respective `GIT` repos. The scripts included in this repo are used to keep everything in sync. These scripts may be used for developing themes as well as plugins.

## Scripts

The scripts in this repository are described as follows:

- **pull-content:** This script will sync the `wp-content/uploads/` directories from your Pantheon `DEV` install to your local environment. As described above, Pantheon omits this directory and the subdirectories contained therein from it's repo (which makes sense, as these can become quite large). This script may be used in both local development environments, as each environment will need this data locally in order to resolve proerly.
- **push-content:** This script will sync the `wp-content/uploads/` directory of your local development environment with the Pantheon `DEV` environment. This script is not used as frequently as `pull-content`, as most of the time one is adding content and files directly to the Pantheon `DEV` environment. This especially true if many people are working on content development for a site.
- **synctheme:** This script will sync the files of your theme or plugin from your local install containing clones of those repos to the local install of the Pantheon repo.
- **rsync-exclude.txt:** this file is similar to a `.gitignore` file but for rsync. It lists all the files and directories you want to exclude when you run the `synctheme` script. At the very least it should include your `.git/` directory and your `.gitignore` file.
