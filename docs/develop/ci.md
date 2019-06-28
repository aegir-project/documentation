Continuous integration
======================

Primary tests are done on [GitLab CI](http://gitlab.com/aegir/provision/pipelines/).
There are also tests on [Travis](https://github.com/aegir-project/tests/).

Setting up a GitLab runner
--------------------------

GitLab.com offers the use of shares runner VM's to use for CI, but we need a priviledged one to be able to use sudo and sub docker images.
As a project we maintain one runner for this, but if you want to experiment more in a feature branch of your own fork then it's best to setup a new one.
Documentation on setting one up is on https://docs.gitlab.com/runner/install/
The minimal memory requirement seems to be 1GB. And up to 10GB of disk storage.

Example config file /etc/gitlab-runner/config.toml:
```
concurrent = 1
check_interval = 0

[[runners]]
  name = "Initfour-helmo-ci_runner01"
  url = "https://gitlab.com/ci"
  token = "9N-J78KC9b7oQE8LMP_d"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "ruby:2.1"
    privileged = true
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
  [runners.cache]
```

The modified lines are 'privileged' and 'volumes'


Troubleshooting CI tests
------------------------

When you have your own GitLab CI runner you can inspect the container while it's running the tests.
Adding an extra `sleep 20m` as a last task in the .gitlab-ci.yml script section would keep it alive a bit longer. The default timeout is 3600 seconds.

Use `docker ps` to get the container ID, and `docker exec -ti [containerID] bash` to get a shell inside the container.

