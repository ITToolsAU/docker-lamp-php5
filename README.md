![Docker-LAMP](https://cdn.rawgit.com/mattrayner/docker-lamp/831976c022782e592b7e2758464b2a9efe3da042/docs/logo.svg)

Docker-LAMP-PHP5 is a Docker image that includes the Phusion base along with a LAMP stack ([Apache 2.4.7](http://www.apache.org/), [MySQL 5.7](https://www.mysql.com/) and [PHP 5.6](http://php.net/)) on Ubuntu 16.04 Xenial, all in one handy container. [phpMyAdmin](https://www.phpmyadmin.net/) is also bundled.

**This image is only intended for legacy PHP 5.6 applications, which is [end-of-life](https://www.php.net/supported-versions.php) as of January 2019. Use at your own risk, preferably *not* in production and/or public-facing environments!**

Based off an old version of [mattrayner/docker-lamp](https://github.com/mattrayner/docker-lamp).


## Usage

### Directory structure

```
/ (project root)
/app/ (your PHP files aka the web root)
/mysql/ (Docker will create this and store your MySQL data here)
```

### Starting from command line

```
docker run -p "80:80" -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql gcr.io/jakejarvis/lamp-php5:latest
```

### Starting with Docker Compose

```
version: "3"
services:
  lamp:
    image: gcr.io/jakejarvis/lamp-php5:latest
    ports:
      - "80:80"
    volumes:
      - "./app:/app"
      - "./mysql:/var/lib/mysql"
```

### Starting from a Dockerfile

```
FROM gcr.io/jakejarvis/lamp-php5:latest

# Your custom commands

CMD ["/run.sh"]
```

### MySQL Databases

When you first run the image, you'll see a message showing your `admin` user's password. This is the user you should use in your application. If you need this login later, you can run `docker logs CONTAINER_ID` and you should see it at the top of the log.

You can access [phpMyAdmin](https://www.phpmyadmin.net/) at `/phpmyadmin` with the `admin` username and password.

By default, the image comes with a `root` MySQL account that has no password. This account is only available locally, i.e. within your application. It is not available from outside your Docker image or through phpMyAdmin.


## License
Docker-LAMP is licensed under the [Apache 2.0 License](LICENSE.md).

To facilitate local tracker debug work, it was added to the main vagrant environment file:

```
config.vm.define "tracker", primary: false do |box|
        box.hostmanager.aliases = [ "tracker."+dev_domain ]
        box.vm.network :private_network, ip: "#{ip_range}.201", subnet: "#{ip_range}.0/16"
        box.vm.hostname = "tracker#{dev_domain}"
        box.ssh.insert_key = true
        box.ssh.username = "vagrant"
        box.ssh.password = "vagrant"
        box.vm.provider 'docker' do |d|
            d.build_dir = "#{vagrant_root}/docker/docker-lamp-php5/"
            d.dockerfile = "Dockerfile"
            d.has_ssh = true
            d.name = "tracker_#{dev_domain}"
            d.create_args = ["--cap-add=NET_ADMIN"]
            d.remains_running = true
            d.volumes = [ENV['HOME']+"/.ssh/:/home/vagrant/.ssh", "#{vagrant_root}/sites/tracker:/app" , "#{vagrant_root}/persistent_storage/tracker_mysql:/var/lib/mysql"]
        end
        ## FINAL BOX MUST HAVE THIS
        box.trigger.after :up do |trigger|
            trigger.run = {inline: "bash -c 'vagrant hostmanager --provider docker'"}
        end
        ##
    end
```

Thsi repo must be cloned into teh docker folder for this to work

