# docker-maven-build
This is a base image for building [java][java] [maven][maven] repositories.

## Purpose
This docker image builds on top of Arch Linux's base/archlinux image for the
purpose of building projects using mvn  It provides several key features:

* A non-root user (`build`) for executing the image build.  This is important
  for security purposes and to ensure that the package doesn't require root
  permissions to be built.
* Access to the build location will be in the volume located at `/code`.  This
  directory will be the default working directory.

## Usage
This library is useful from the command line.  For example:

```bash
docker run --interactive --tty --rm --volume /tmp/my-code:/code nubs/maven-build

# Using short-options:
# docker run -i -t --rm -v /tmp/my-code:/code nubs/maven-build
```

This will execute the default command (`mvn`) and update your code directory
with the result.

Other commands can also be executed.  For example, to run different tasks:

```bash
docker run -i -t --rm -v /tmp/my-code:/code nubs/maven-build mvn install
```

## Permissions
This image uses a build user to run mvn.  This means that your file permissions
must allow this user to write to certain folders.  The easiest way to do this
is to create a group and give that group write access to the necessary folders.

```bash
groupadd --gid 59944 mvn-build
chmod -R g+w node_modules
chgrp -R mvn-build node_modules
```

You may also want to give your user access to files created by the build user.

```bash
usermod -a -G 51177 "$(whoami)"
```

### Dockerfile build
Alternatively, you can create your own `Dockerfile` that builds on top of this
image.  This allows you to modify the environment by installing additional
software needed, altering the commands to run, etc.

A simple one that just installs another package but leaves the rest of the
process alone could look like this:

```dockerfile
FROM nubs/maven-build

USER root

RUN pacman --sync --noconfirm --noprogressbar --quiet somepackage

USER build
```

You can then build this docker image and run it against your project volume
like normal (this example assumes the `Dockerfile` are in your current
directory):

```bash
docker build --tag my-code .
docker run -i -t --rm -v "$(pwd):/code" my-code
docker run -i -t --rm -v "$(pwd):/code" my-code mvn install
```

[java]: http://java.com/
[maven]: https://maven.apache.org/
