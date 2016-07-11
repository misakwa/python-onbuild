WHY
===
When working on so many different python versions for projects, it becomes
tedius repeating these commands to compile python from source.
The aim of this little project here is to allow one to build a project simply by
extending from one central image and build for a various python versions using build arguments.
This work borrows heavily from https://github.com/docker-library/python.

Find versions here: https://www.python.org/downloads/source/


USAGE EXAMPLE
=============
Extend from another Dockerfile and specify PYTHON_VERSION on build

```
FROM quay.io/misakwa/python-onbuild:alpine-3.4

MAINTAINER you<you@example.com>

COPY . /opt/code/

# install dependencies for project
RUN ...

CMD ["some_command_exposed_by_project"]
```

```sh
$ docker build --build-arg PYTHON_VERSION=3.5.0 -t myimage:1.0-python3.5 --rm --force-rm .
$ # Open up a shell
$ docker run -it --rm myimage:1.0-python3.5 sh
```

TODO
====
- Use preprocessing to share code and reduce duplication
- Implement other OSes
- Contribute back to https://github.com/docker-library/python
