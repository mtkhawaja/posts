# Docker Compose ENV Files and Portainer

In addition to the default [.env file](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with-an-env-file) located in the project root, we can specify env files as follows:

1. Using the `--env-file` CLI option
2. By defining the `env_file` attribute in the docker compose file itself

## env_file vs --env-file

Regrading the `env_file` attribute, the [docker documentation](https://docs.docker.com/compose/environment-variables/set-environment-variables/#use-the-env_file-attribute) states:

> With this option, environment variables declared in the file cannot then be referenced again separately in the Compose file or used to configure Compose.

In essence, the `--env-file` option can be used to make key-value pairs available to **both** the compose file itself and also to the container.
In contrast, the `env_file` can **only** be used to make key-value pairs available as environment variables to the container.

The contents of the `common.env` referred to in the following sections are as follows:

```text
BASE_VOLUME_DIRECTORY=/mnt/primary-storage/docker-volumes/
FIRST_PROGRAM=HELLO-WORLD
```

### --env-file

Consider the following docker compose file which will print out the available environment variables in the container:

```yaml
version: "3.9"
services:
  demo:
    image: alpine:latest
    command: [ "sh", "-c", "env" ]
    environment:
      - FIRST_PROGRAM
      - SECOND_PROGRAM=${FIRST_PROGRAM}
      - BASE_VOLUME_DIRECTORY=${BASE_VOLUME_DIRECTORY}
```

When using the `--env-file` option, the key-value pairs will be available to the compose file itself.

Consequently, the variables defined in the `enviroment` section will be set based on the contents of the env file and thus will be made available to the container as well.

Run the container as follows:

```bash
#!/usr/bin/env bash

docker compose --env-file=/path/to/common.env up

```

The output will include the following

```text
...
FIRST_PROGRAM=HELLO-WORLD
SECOND_PROGRAM=HELLO-WORLD
BASE_VOLUME_DIRECTORY=/mnt/primary-storage/docker-volumes
...
```

### env_file

On the other hand, if you specify an env file via the `env_file` attribute as follows:

```yaml
version: "3.9"
services:
  demo:
    image: alpine:latest
    env_file:
      - /path/to/common.env
    command: [ "sh", "-c", "env" ]
    environment:
      - SECOND_PROGRAM=${FIRST_PROGRAM}
```

Then the environment variable `SECOND_PROGRAM` will be set to empty as `FIRST_PROGRAM` is not available to the compose file itself while `BASE_VOLUME_DIRECTORY` & `FIRST_PROGRAM` will be available to the container as expected.

When you run the container:

```bash
#!/usr/bin/env bash

docker compose up
```

Docker will first display a warning message for `FIRST_PROGRAM` :

```text
WARN[0000] The "FIRST_PROGRAM" variable is not set. Defaulting to a blank string.
```

Recall however that while `FIRST_PROGRAM` is not available to the compose file it will still be available to the container.

As such, the container will show an empty value for `SECOND_PROGRAM` while `FIRST_PROGRAM` & `BASE_VOLUME_DIRECTORY` will be set as expected i.e.

```text
...
FIRST_PROGRAM=HELLO-WORLD
SECOND_PROGRAM=
BASE_VOLUME_DIRECTORY=/mnt/primary-storage/docker-volumes
...
```

## Portainer

### Global Environment Variables and Using the env_file attribute

Suppose that there are some environment variables that are common to all the docker compose files in your setup. For instance, the base path on the host for bind mounts might be the same everywhere.

Typically, the `--env-file` option could be used to point to an env file containing all the common key-value pairs. However, if we run the portainer container with the environment variables specified, then those environment variables will be available to any docker compose based [portainer stack definition](https://docs.portainer.io/user/docker/stacks):

Additionally, to use the `env_file` attribute in the stack definitions we will have to specify the **container path** for the env files which will require mounting the directory containing the env files.

Given the docker compose file for portainer below:

```yaml
# See Portainer Community Edition Installation for more details: https://docs.portainer.io/start/install-ce/server/docker/linux
version: "3.9"
services:
  portainer:
    image: "portainer/portainer-ce:latest"
    container_name: "portainer"
    ports:
      - "8000:8000"
      - "9443:9443"
    restart: "unless-stopped"
    security_opt:
      - no-new-privileges:true
    environment:
      # Specify any environment variables that you want to be available to all stacks here
      - FIRST_PROGRAM=${FIRST_PROGRAM}
      - BASE_VOLUME_DIRECTORY=${BASE_VOLUME_DIRECTORY}
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "$BASE_VOLUME_DIRECTORY/env-files:/env-files:ro"
      - "$BASE_VOLUME_DIRECTORY/portainer/data:/data"
```

We can run the portainer container as follows:

```bash
!/usr/bin/env bash

docker compose --env-file=/path/to/common.env up -d
```

We can verify that everything is working as expected by creating a stack with the following definition:

```yaml
version: "3.9"
services:
  demo:
    image: alpine:latest
    env_file:
      - /env-files/demo.env
    environment:
      - FIRST_PROGRAM
      - SECOND_PROGRAM=${FIRST_PROGRAM}
      - BASE_VOLUME_DIRECTORY=${BASE_VOLUME_DIRECTORY}
    command: [ "sh", "-c", "env" ]
```

Assume that another env file `demo.env` exists in the `/env-files` directory as follows:

```text
DEMO_TITLE=ENVIRONMENT-VARIABLES-DEMO
```

We should see the following log output:

```text
...
FIRST_PROGRAM=HELLO-WORLD
SECOND_PROGRAM=HELLO-WORLD
DEMO_TITLE=ENVIRONMENT-VARIABLES-DEMO
BASE_VOLUME_DIRECTORY=/mnt/primary-storage/docker-volumes
...
```

## References / Additional Reading

1. “Environment Variables Precedence in Docker Compose.” Docker Documentation, [docs.docker.com/compose/environment-variables/envvars-precedence/](https://docs.docker.com/compose/environment-variables/envvars-precedence/).
2. “Stacks = Docker-Compose, the Portainer Way.”, Portainer, [www.portainer.io/blog/stacks-docker-compose-the-portainer-way](https://www.portainer.io/blog/stacks-docker-compose-the-portainer-way).
3. “Using Env Files in Stacks with Portainer.”, Portainer, [www.portainer.io/blog/using-env-files-in-stacks-with-portainer](https://www.portainer.io/blog/using-env-files-in-stacks-with-portainer).
4. “Ways to Set Environment Variables in Compose.” Docker Documentation, [docs.docker.com/compose/environment-variables/set-environment-variables](https://docs.docker.com/compose/environment-variables/set-environment-variables)