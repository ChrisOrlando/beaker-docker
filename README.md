# beaker-docker

Beaker library to use docker hypervisor

## How to use this wizardry

This gem that allows you to use hosts with [docker](docker.md) hypervisor with [beaker](https://github.com/voxpupuli/beaker).

Beaker will automatically load the appropriate hypervisors for any given hosts
file, so as long as your project dependencies are satisfied there's nothing else
to do. No need to `require` this library in your tests.

In order to use a specific hypervisor or DSL extension library in your project,
you will need to include them alongside Beaker in your Gemfile or
project.gemspec. E.g.

```ruby
# Gemfile
gem 'beaker', '~> 4.0'
gem 'beaker-docker'
# project.gemspec
s.add_runtime_dependency 'beaker', '~> 4.0'
s.add_runtime_dependency 'beaker-docker'
```

### Nodeset Options

The following is a sample nodeset:

```yaml
HOSTS:
  el8:
    platform: el-8-x86_64
    hypervisor: docker
    image: centos:8
    docker_cmd: '["/sbin/init"]'
    # Run arbitrary things
    docker_image_commands:
      - 'touch /tmp/myfile'
    dockeropts:
      Labels:
        thing: 'stuff'
      HostConfig:
        Privileged: true
  el7:
    platform: el-7-x86_64
    hypervisor: docker
    image: centos:7
    # EL7 images do not support nested systemd
    docker_cmd: '/usr/sbin/sshd -D -E /var/log/sshd.log'
CONFIG:
  docker_cap_add:
    - AUDIT_WRITE
```

### Privileged containers

Containers are run in privileged mode by default unless capabilities are set.

If you wish to disable privileged mode, simply set the following in your node:

```yaml
dockeropts:
  HostConfig:
    Privileged: false
```

### Cleaning up after tests

Containers created by this plugin may not be destroyed unless the tests complete
successfully. Each container created is prefixed by `beaker-` to make filtering
for clean up easier.

A quick way to clean up all nodes is as follows:

```sh
podman rm -f $( podman ps -q -f name="beaker-*" )
```

## Working with `podman`

If you're using a version of `podman` that has API socket support then you
should be able to simply set `DOCKER_HOST` to your socket and connect as usual.

You also need to ensure that you're using a version of the `docker-api` gem that
supports `podman`.

You may find that not all of your tests work as expected. This will be due to
the tighter system restrictions placed on containers by `podman`. You may need
to edit the `dockeropts` hash in your nodeset to include different flags in the
`HostConfig` section.

See the
[HostConfig](https://any-api.com/docker_com/engine/docs/Definitions/HostConfig)
portion of the docker API for more information.

## Spec tests

Spec test live under the `spec` folder. There are the default rake task and therefore can run with a simple command:

```bash
bundle exec rake test:spec
```

## Acceptance tests

There is a simple rake task to invoke acceptance test for the library:

```bash
bundle exec rake test:acceptance
```

## Contributing

Please refer to puppetlabs/beaker's [contributing](https://github.com/puppetlabs/beaker/blob/master/CONTRIBUTING.md) guide.

## Releasing

To release new versions of beaker-docker, please use update `lib/beaker-docker/version.rb` with the new version number. The version should adhere to [semantic version standards](https://semver.org). When in doubt, ask in the `#voxpupuli` channel of the Puppet community Slack or in `#voxpupuli` on irc.freenode.net ([Webinterface](https://webchat.freenode.net/?channels=%23voxpupuli)).

Once the version file has been updated, a contributor can push a new git tag to match and GitHub Actions will do the release to RubyGems.
