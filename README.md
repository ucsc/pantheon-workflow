# A Development workflow for Pantheon.io

[Pantheon.io](https://pantheon.io) is a robust [WordPress](https://wordpress.org) and [Drupal](https://drupal.org) hosting service that provides three environments per site: `DEV`, `TEST` and `LIVE`. Pantheon enforces a workflow whereby one "develops" on `DEV`, "tests" on `TEST` and you point your domain name to `LIVE`. In order to update a site, one updates `DEV` (WP update, plugin update, etc.). In order to test the update, Pantheon pushes the new code from `DEV` and pulls the database from `LIVE` into `TEST` in order to test the update. If all tests well, the new code is the pushed to `LIVE`. 

Pantheon also maintains `GIT` repos of each of these environments, which a developer may pull from and push to when developing in an local development environment such as WAMPP\MAMPP\LAMPP. (While all three environments are available via a `GIT` repo, it is _highly_ recommended that a developer only utilize the `DEV` repo.)

From a local development standpoint, a significant drawback of this setup is that the Pantheon repositories are comprised of the _entire_ WordPress (or Drupal) install. (_Almost_ the entire install -- Pantheon excludes the uploads directory, eg. `wp-content/uploads/` in WordPress -- more on that below.)

The reason that this is a drawback is because this configuration does not allow a developer to develop a theme or plugin in a separate repository such as [Github](https://github.com). Pantheon will not permit the creation or cloning of additional repositories _within_ the larger repository of one's development site.


