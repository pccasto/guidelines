# Docker

Running MoOde, or moOde processes in a docker container may allow for quicker
testing, but is not part of a current (or planned) workflow.

This document exists to capture any experimental efforts that may be taken.

Possible starting points:

- https://ownyourbits.com/2018/06/13/transparently-running-binaries-from-any-architecture-in-linux-with-qemu-and-binfmt_misc/
- https://ownyourbits.com/2018/06/27/running-and-building-arm-docker-containers-in-x86/

Much of the rest of this is carry-over from a project that does use docker, and
is left here in the hopes that it one day may be useful.


First, make sure you [prepared your OS](os-setup/README.md).
Then [set up your project](project-setup/README.md) to use our Docker configuration.

In case you run into problems, refer to the [troubleshooting chapter](troubleshooting/README.md).



## Start your project

- `cd` into to the project directory.
- Run `docker-compose up`
- Now you can run the site on [localhost](http://localhost), optionally followed by a port number if you have defined one in `docker-compose.yml`.



## Stop your project

Use `CTRL + C` in a running Docker window to stop both.
