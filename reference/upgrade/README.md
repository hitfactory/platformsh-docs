# Upgrading

## Changes in version 2015.7
In the `.platform.app.yaml` configuration file we now allow for a much clearer syntax, which you can (and should) start using now.

The old format had a single string to identify the 'toolstack' you use:
```yaml
toolstack: "php:drupal"
```

The new syntax allows to separate the concerns of what language you are running
and the build process that is going to happen on deployment:
```yaml
type: php
build:
    flavor: drupal
```
Currently we only support `php` in the 'type' key. Current supported build
flavors are `drupal`, `composer` and `symfony`.

## Changes in version 1.7.0

Version 1.7.0, deployed on September 4th, 2014, introduces changes in
the configuration files format. Most of the old configuration format is
still supported, but customers are invited to move to the new format.

For an example upgrade path, see the [Drupal 7.x branch of the
Platformsh-examples
repository](https://github.com/platformsh/platformsh-examples/blob/drupal/7.x/.platform.app.yaml)
on GitHub.

Configuration items for PHP that previously was part of
`.platform/services.yaml` are now moved into `.platform.app.yaml`, which
gains the following top-level items:

-   `name`: should be `"php"`
-   `relationships`, `access` and `disk`: should be the same as the
    `relationships` key of PHP in `.platform/services.yaml`

Note that there is now a sane default for `access` (SSH access to PHP is
granted to all users that have role "collaborator" and above on the
environment) so most customers can now just omit this key in
`.platform.app.yaml`.

In addition, version 1.7.0 now has consistency checks for configuration
files and will reject `git push` operations that contain configuration
files that are invalid. In this case, just fix the issues as they are
reported, commit and push again.

