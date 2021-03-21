---
title: "Running GitLab and GitLab Runner within Docker"
tags: [CI, Docker, GitLab, GitLab Runner]
---

## Running GitLab and GitLab Runner within Docker

This is how I configured GitLab and GitLab Runner on a Docker host. The runner is configured to use the Docker instance on the host for the runners it creates.

1. Install CentOS
1. Install Docker using [Docker instructions](https://docs.docker.com/engine/installation/linux/centos/)
1. Create network for GitLab containers

    ```bash
    sudo docker network create --driver=bridge gitlab-net
    ```

1. Create volumes for GitLab

    ```bash
    sudo docker volume create --name=gitlab-config
    sudo docker volume create --name=gitlab-data
    sudo docker volume create --name=gitlab-logs
    ```

1. Create GitLab container (SSH port has to be mapped to a different, non-standard port. Port 22 is being used by the CentOS host)

    ```bash
    sudo docker run --detach --hostname gitlab --net gitlab-net --publish 443:443 --publish 80:80 --publish 23:22 --name gitlab --restart unless-stopped --volume gitlab-config:/etc/gitlab --volume gitlab-logs:/var/log/gitlab --volume gitlab-data:/var/opt/gitlab gitlab/gitlab-ce:latest
    ```

1. Log in to GitLab, set password, log in as root / set password
1. Create volume for GitLab Runner

    Use `-v /var/run/docker.sock:/var/run/docker.sock` to allow container to communicate with the docker instance on the host.

    ```bash
    sudo docker run --detach --hostname gitlab-runner --net gitlab-net --name gitlab-runner --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest
    ```

1. Register runner with GitLab. Coordinator URL is hostname of GitLab container. Token is obtained from the Admin Area (Overview > Runners). Indented lines are prompts.

    ```bash
    sudo docker exec -it gitlab-runner gitlab-runner register</strong>
        Running in system-mode.

        Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
    http://gitlab/
        Please enter the gitlab-ci token for this runner:
    TOKEN_GOES_HERE
        Please enter the gitlab-ci description for this runner:
    [gitlab-runner]:
        Please enter the gitlab-ci tags for this runner (comma separated):

        Registering runner... succeeded runner=******
        Please enter the executor: docker, parallels, ssh, virtualbox, docker+machine, docker-ssh+machine, docker-ssh, shell, kubernetes:
    docker
        Please enter the default Docker image (e.g. ruby:2.1):
    ubuntu
        Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
    ```

1. Configure runners to be created on GitLab docker network

    ```bash
    sudo docker exec -it gitlab-runner /bin/bash
    vi /etc/gitlab-runner/config.toml
    ```

    Add network_mode = "gitlab-net" to config

    ```ini
    concurrent = 1
    check_interval = 0

    [[runners]]
        name = "gitlab-runner"
        url = "http://gitlab/"
        token = "7bef9f69a72d724b45b5401bd848d5"
        executor = "docker"
    [runners.docker]
        tls_verify = false
        image = "ubuntu"
        privileged = false
        disable_cache = false
        volumes = ["/cache"]
        network_mode = "gitlab-net"    <<< Add this line
    [runners.cache]
    ```

    Exit shell on gitlab-runner

1. Create project, create a commit with .gitlab-ci.yml in root. The following CI script runs a build which simply writes the hostname of the runner

    ```yml
    image: ubuntu
    build:
    script:
        - hostname
    ```

1. Run a build from Pipelines in the Project views of GitLab
