# blendle/drone-ci

Simple configuration files to bootstrap a [Drone CI][] environment on your local
Vagrant VM, or any other cloud hoster like [Digital Ocean][].

[Drone CI]: https://github.com/drone/drone
[Digital Ocean]: https://www.digitalocean.com

## Instructions

### 1. Modify `user-data.yml` to your liking

Change anything you want, but be sure to change the following values:

* `coreos.etc.discovery` - get a new key at https://discovery.etcd.io/new

* `write_files[0]` - change the `drone.toml` file to match your desired Drone
  configuration.

### 2. Test drive your setup locally

Make sure you have Vagrant and VirtualBox installed.

Then simply run `vagrant up --provision` to get a working Drone CI environment.

After bootstrapping, visit http://localhost:8080 to see your running
application.

### 3. Run Drone CI in production

Using a host like Digital Ocean, create a new CoreOS VM, and copy/paste your
`user-data.yml` contents to the "user-data" box.

Be sure to enable "Private Networking" and "Enable User Data".

Also, you will have to generate a new etcd dicovery key if you used the previous
one while setting up the environment locally with Vagrant.

Create the VM and log into the machine.

Type the following commands to get a working Drone CI environment:

```bash
$ fleetctl load /etc/systemd/system/drone.service
$ fleetctl start drone.service
```

You can optionally load the "Orphaned Docker Volumes Cleaner" service. This
timer-based service removes any orphaned docker volumes every night at 02:00.

You need to use this if you ever expect to use a docker in docker solution like
[jpetazzo/dind](https://github.com/jpetazzo/dind) to run your own Docker
containers inside a Drone job:

```bash
$ fleetctl load /etc/systemd/system/orphaned-docker-volumes-cleaner.*
$ fleetctl start /etc/systemd/system/orphaned-docker-volumes-cleaner.timer
```

## License

MIT - see the accompanying [LICENSE](LICENSE) file for details.
