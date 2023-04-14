# Containerizing Jenkins, DooD and DinD

**Note**: The sections below use the latest version of
the [Jenkins image](https://hub.docker.com/r/jenkins/jenkins/tags) i.e. `jenkins/jenkins:latest-jdk17`. You should lock
this version down.

## Basic Configuration

The goal is to run a Jenkins container without the requirement of creating any additional containers from Jenkins.

Assuming that [basic](./basic) is the working directory:

```bash
#!/usr/bin/env bash

docker volume create jenkins-home
docker network create jenkins
docker run \
  --name jenkins \
  --restart unless-stopped \
  --detach \
  --network jenkins \
  --publish mode=ingress,published=80,target=8080 \
  --env JAVA_OPTS="-Djava.awt.headless=true" \
  -v jenkins-home:/var/jenkins_home \
  jenkins/jenkins:latest-jdk17

```

Alternatively, see [docker-compose.yaml](./basic/docker-compose.yaml) to start containers using docker compose:

```shell
#!/usr/bin/env bash

docker compose up -d

```

---

## Executing docker commands from a Jenkins container

Being able to run docker commands is useful for situations when Docker images need to be built or dockerized tests such
as those based on [test containers](https://www.testcontainers.org) need to be executed.

There are two approaches to configure a containerized Jenkins to communicate with docker:

---

### Docker Outside of Docker (DooD)

- The basic idea is that containers talk to the Docker Daemon/Server process running on the host system.
- To enable this, the `/var/run/docker.sock` UNIX socket is mounted against a container to allow the container to
  communicate with the Docker daemon on the host.
- According to
  the [docker documentation](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option.):

> By default, a unix domain socket (or IPC socket) is created at /var/run/docker.sock, requiring either root permission,
> or docker group membership.

- Essentially, the Docker daemon listens on the `/var/run/docker.sock` UNIX socket. This is used by Docker clients
  to communicate with the server process.

We'll build a custom Jenkins Image with docker installed. See [Dockerfile](./dood/Dockerfile) for the custom Dockerfile.
We're adding docker to the base Jenkins image.

And then Assuming [dood](./dood) is the working directory:

```shell
#!/usr/bin/env bash

docker volume create jenkins-home
docker network create jenkins
docker build -t custom/jenkins-with-docker ./

docker run \
  --name jenkins \
  --restart unless-stopped \
  --detach \
  --network jenkins \
  --publish mode=ingress,published=80,target=8080 \
  --env JAVA_OPTS="-Djava.awt.headless=true" \
  -v jenkins-home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  custom/jenkins-with-docker

```

Alternatively, see [docker-compose.yaml](./dood/docker-compose.yaml) to start containers using docker compose:

```shell
#!/usr/bin/env bash

docker compose up -d
```

#### Consequences of DooD

- Any containers started by the Jenkins containers will be its siblings and visible to the host system.

---

### Docker inside of Docker (DinD)

- The basic idea is that docker itself is started as a [container](https://hub.docker.com/_/docker) and networked with
  the Jenkins container.

We'll build a custom Jenkins Image with docker installed. See [Dockerfile](./dind/Dockerfile) for the custom Dockerfile.
We're adding docker to the base Jenkins image.

Assuming [dind](./dind) is the working directory:

```shell
#!/usr/bin/env bash
docker network create jenkins
docker volume create jenkins-home
docker volume create jenkins-docker-certs
# Start the DinD (Docker in Docker) Container
docker run \
  --name ci-cd-dind \
  --restart unless-stopped \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --publish mode=ingress,published=2376,target=2376 \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-home:/var/jenkins_home \
  docker:dind
# Start the Jenkins Container
docker build -t custom/jenkins-with-docker ./
docker run \
  --name jenkins \
  --restart unless-stopped \
  --detach \
  --network jenkins \
  --publish mode=ingress,published=80,target=8080 \
  --env JAVA_OPTS="-Djava.awt.headless=true" \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  -v jenkins-home:/var/jenkins_home \
  -v jenkins-docker-certs:/certs/client:ro \
  custom/jenkins-with-docker

```

See [docker-compose.yaml](./dind/docker-compose.yaml) for compose based dood configuration.

```shell
#!/usr/bin/env bash

docker compose up -d

```

#### Consequences of DinD

- Any containers started by the containers are its **children** and not visible to the host system.
- The privileged flag is required for DinD. As per
  the [docker documentation](https://docs.docker.com/engine/reference/commandline/run/#privileged):

> --privileged flag gives all capabilities to the container, and it also lifts all the limitations enforced by the
> device cgroup controller. In other words, the container can then do almost everything that the host can do. This flag
> exists to allow special use-cases, like running Docker within Docker.

In other words, if the container gets compromised, the host gets compromised as well.

---

### Jenkins Configuration and Docker Validation

Regardless of whether DinD or DooD was used, you'll want to install a couple of plugins to use Docker i.e.

- [docker-plugin](https://plugins.jenkins.io/docker-plugin/)
- [docker-pipeline-plugin](https://plugins.jenkins.io/docker-workflow/)

After installing the plugins, you can create a pipeline job to verify that everything is working:

```groovy
pipeline {
    agent {
        docker { image 'node:latest' }
    }
    stages {
        stage('Verify Docker Integration') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

#### Verifying DooD Sibling Relationship

Assuming you hadn't already pulled the latest node image, running the `docker image ls` command on the host will show
you the node image in the image listing.

```text
# docker image ls output
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
node         latest    5bb57e984682   42 hours ago   999MB
....
```

#### Verifying DinD Parent/Child Relationship

Assuming you hadn't already pulled the latest node image, running the `docker image ls` command on the host **will not
show the latest node image in the listing**. However, if you connect to the jenkins container and check the listing
there it will show up in the image listing:

```shell
#!/usr/bin/env bash

docker container exec -it jenkins bash
# From within the container
docker image ls
```

```text
# docker image ls output
root@d1ffb64a9278:/# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
node         latest    5bb57e984682   43 hours ago   999MB
```

---

## References / Additional Reading

- "A Case for Docker-in-Docker on Kubernetes",
  applatix, [https://applatix.com/case-docker-docker-kubernetes-part](https://applatix.com/case-docker-docker-kubernetes-part)
- "Do not use docker in docker for ci",
  jpetazzo, [https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
- "Docker Architecture",
  Docker, [https://docs.docker.com/get-started/overview/#docker-architecture](https://docs.docker.com/get-started/overview/#docker-architecture)
- "Docker",
  Jenkins, [https://www.jenkins.io/doc/book/installing/docker/](https://www.jenkins.io/doc/book/installing/docker/)
- “Dockerd; Daemon Socket Option”, Docker
  Documentation, [https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option).
- "Dockerhub Jenkins Image",
  DockerHub, [https://hub.docker.com/r/jenkins/jenkins/tags](https://hub.docker.com/r/jenkins/jenkins/tags)
- "Enhanced Container Isolation (ECI)",
  Docker, [https://docs.docker.com/desktop/hardened-desktop/enhanced-container-isolation/how-eci-works](https://docs.docker.com/desktop/hardened-desktop/enhanced-container-isolation/how-eci-works/)
- “How to Add Java Arguments to Jenkins?”, CloudBees
  Documentation, [https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-add-java-arguments-to-jenkins#_running_jenkins_inside_docker](https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-masters/how-to-add-java-arguments-to-jenkins#_running_jenkins_inside_docker).
- "How to Setup Docker Containers As Build Agents for Jenkins",
  CloudBeesTV, [https://www.youtube.com/watch?v=ymI02j-hqpU](https://www.youtube.com/watch?v=ymI02j-hqpU)
- "Privileged Flag",
  Docker, [https://docs.docker.com/engine/reference/commandline/run/#privileged](https://docs.docker.com/engine/reference/commandline/run/#privileged)
- "Rootless Containers",
  Docker, [https://docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/)
- "Sysbox", nestybox, [https://github.com/nestybox/sysbox](https://github.com/nestybox/sysbox)
- "What is the purpose of docker-in-docker when using a dockerized Jenkins?", Jenkins
  Community, [https://community.jenkins.io/t/what-is-the-purpose-of-docker-in-docker-when-using-a-dockerized-jenkins/1370/1](https://community.jenkins.io/t/what-is-the-purpose-of-docker-in-docker-when-using-a-dockerized-jenkins/1370/1)
- "Why running privileged containers is a bad idea",
  trendmicro, [https://www.trendmicro.com/en_us/research/19/l/why-running-a-privileged-container-in-docker-is-a-bad-idea.html](https://www.trendmicro.com/en_us/research/19/l/why-running-a-privileged-container-in-docker-is-a-bad-idea.html)
- "var run docker.sock",
  Educative, [https://www.educative.io/answers/var-run-dockersock](https://www.educative.io/answers/var-run-dockersock)

## Attributions

- "Jenkins is the way",
  Jenkins, [https://www.jenkins.io](https://www.jenkins.io/images/logos/jenkins-is-the-way/jenkins-is-the-way.png),
  licensed under
  the [Creative Commons Attribution-ShareAlike 3.0 Unported License](https://creativecommons.org/licenses/by-sa/3.0/) 