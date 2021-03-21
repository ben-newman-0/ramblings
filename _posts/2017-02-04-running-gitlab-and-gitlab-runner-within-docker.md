---
title: "Running GitLab and GitLab Runner within Docker"
tags: [CI, Docker, GitLab, GitLab Runner]
---

This is how I configured GitLab and GitLab Runner on a Docker host. The runner is configured to use the Docker instance on the host for the runners it creates.
<ol>
 	<li>Install CentOS</li>
 	<li>Install Docker using <a href="https://docs.docker.com/engine/installation/linux/centos/">Docker instructions</a></li>
 	<li>Create network for GitLab containers
<ol>
 	<li>
<pre>sudo docker network create --driver=bridge gitlab-net</pre>
</li>
</ol>
</li>
 	<li>Create volumes for GitLab
<ol>
 	<li>
<pre>sudo docker volume create --name=gitlab-config</pre>
</li>
 	<li>
<pre>sudo docker volume create --name=gitlab-data</pre>
</li>
 	<li>
<pre>sudo docker volume create --name=gitlab-logs</pre>
</li>
</ol>
</li>
 	<li>Create GitLab container (SSH port has to be mapped to a different, non-standard port. Port 22 is being used by the CentOS host)
<ol>
 	<li>
<pre>sudo docker run --detach --hostname gitlab --net gitlab-net --publish 443:443 --publish 80:80 --publish 23:22 --name gitlab --restart unless-stopped --volume gitlab-config:/etc/gitlab --volume gitlab-logs:/var/log/gitlab --volume gitlab-data:/var/opt/gitlab gitlab/gitlab-ce:latest</pre>
</li>
</ol>
</li>
 	<li>Log in to GitLab, set password, log in as root / set password</li>
 	<li>Create volume for GitLab Runner
<ol>
 	<li>Use -v /var/run/docker.sock:/var/run/docker.sock to allow container to communicate with the docker instance on the host.
<pre>sudo docker run --detach --hostname gitlab-runner --net gitlab-net --name gitlab-runner --restart unless-stopped -v /var/run/docker.sock:/var/run/docker.sock -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:latest</pre>
</li>
</ol>
</li>
 	<li>Register runner with GitLab. Coordinator URL is hostname of GitLab container. Token is obtained from the Admin Area (Overview &gt; Runners). Bold text is entered, rest are prompts.
<ol>
 	<li>
<pre><strong>sudo docker exec -it gitlab-runner gitlab-runner register</strong>
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
<strong>http://gitlab/</strong>
Please enter the gitlab-ci token for this runner:
<strong>TOKEN_GOES_HERE</strong>
Please enter the gitlab-ci description for this runner:
[gitlab-runner]:
Please enter the gitlab-ci tags for this runner (comma separated):

Registering runner... succeeded runner=******
Please enter the executor: docker, parallels, ssh, virtualbox, docker+machine, docker-ssh+machine, docker-ssh, shell, kubernetes:
<strong>docker</strong>
Please enter the default Docker image (e.g. ruby:2.1):
<strong>ubuntu</strong>
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!</pre>
</li>
</ol>
</li>
 	<li>Configure runners to be created on GitLab docker network
<ol>
 	<li>
<pre>sudo docker exec -it gitlab-runner /bin/bash</pre>
</li>
 	<li>
<pre>vi /etc/gitlab-runner/config.toml</pre>
</li>
 	<li>Add network_mode = "gitlab-net" to config (highlighted in bold)
<pre>concurrent = 1
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
<strong> network_mode = "gitlab-net"</strong>
 [runners.cache]</pre>
</li>
 	<li>Exit shell on gitlab-runner</li>
</ol>
</li>
 	<li>Create project, create a commit with .gitlab-ci.yml in root. The following CI script runs a build which simply writes the hostname of the runner
<ol>
 	<li>
<pre class="code highlight"><span id="LC1" class="line"><span class="na">image</span><span class="pi">:</span> <span class="s">ubuntu

</span></span><span id="LC2" class="line"></span><span id="LC3" class="line"><span class="na">build</span><span class="pi">:
</span></span><span id="LC4" class="line"><span class="na">  script</span><span class="pi">:
</span></span><span id="LC5" class="line"><span class="pi">    -</span> <span class="s">hostname</span></span></pre>
</li>
</ol>
</li>
 	<li>Run a build from Pipelines in the Project views of GitLab</li>
</ol>
