# NCS + Docker Example Application

This repository contains instructions on how to use Docker container as a
development build environment when creating NSC applications.

It consists of:
* a fork of Zephyr's Out-of-tree example.
* `docker_scripts` folder

This repository is suitable for building v1.8.0 NCS applications.
The Out-of-tree example was modified to so that it could be built against
v1.8.0 NCS, namely:
* Zephyr include paths had to be corrected and
* `west.yaml` was changed so it pulls in only NCS repo.

## Getting Started

### Repo initialization

The first step is to initialize the workspace folder (``my-workspace``) where
the ``example-application`` and all Zephyr modules will be cloned. You can do
that by running:

```bash
west init -m https://github.com/MarkoSagadin/ncs-docker-setup my-workspace
cd my-workspace
west update
cd example-application
```

#### Install Docker

If you do not have Docker installed on your system run:
```bash
cd docker_scripts
./docker_install
```

Above needs to be done only once.

#### Build Docker image

Next you will need to build the Docker image, this will take some time:

```bash
cd docker_scripts
./docker_build
```

This needs to be done only once for this project.

### Development cycle

To run the Docker container which contains all required tools:

```bash
cd docker_scripts
./docker_run
```

You will login in home directory, your project is located inside of
`ncs-project` directory.


When you are done with your work you can close the terminal or exit from the
container with:
```bash
exit
```

All changes that are done inside of `ncs-project` are shared with your NCS
project on you host machine.

Once inside the container you can use `west` as usual.

#### Build and run

The application can be built by running:

```bash
west build -b $BOARD -s app
```

where `$BOARD` is the target board. The `custom_plank` board found in this
repository can be used. Note that Zephyr sample boards may be used if an
appropriate overlay is provided (see `app/boards`).

A sample debug configuration is also provided. You can apply it by running:

```bash
west build -b $BOARD -s app -- -DOVERLAY_CONFIG=debug.conf
```

Note that you may also use it together with `rtt.conf` if using Segger RTT. Once
you have built the application you can flash it by running:

```bash
west flash
```
