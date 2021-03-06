# HHVM Beta Support

HHVM is an open-source virtual machine designed for executing programs written in Hack and PHP. HHVM uses a just-in-time (JIT) compilation approach to achieve superior performance while maintaining the development flexibility that PHP provides.

Note that there are some differences between PHP and HHVM and you should really test your application before deploying with
HHVM. 

HHVM can be much, much faster then PHP so it is worth the trouble of adjusting your code for it.

To switch your project to HHVM, put `hhvm` instead of `php` in the `type`
property of your `.platform.app.yaml`.

Example:

```yaml
    name: "fastapp"
    type: hhvm:3.10
    build:
        flavor: composer
    web:
      document_root: "/"
      passthru: "/index.php"
    disk: 2048
```
