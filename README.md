# Unbound DNS Server Docker Image

Run Unbound with latest version of OpenSSL on Raspberry Pi with Docker.

Do you want this, but don't have a Pi? Check out the [non-ARM version](https://github.com/MatthewVance/unbound-docker).

## Supported tags and respective `Dockerfile` links
- [`1.21.1`, `latest` (*1.21.1/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.21.1)
- [`1.21.0`, (*1.21.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.21.0)
- [`1.20.0`, (*1.20.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.20.0)
- [`1.19.3`, (*1.19.3/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.19.3)
- [`1.19.2`, (*1.19.2/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.19.2)
- [`1.19.1`, (*1.19.1/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.19.1)
- [`1.19.0`, (*1.19.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.19.0)
- [`1.18.0`, (*1.18.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.18.0)
- [`1.17.1`, (*1.17.1/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.17.1)
- [`1.17.0`, (*1.17.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.17.0)
- [`1.16.3`, (*1.16.3/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.16.3)
- [`1.16.2`, (*1.16.2/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.16.2)
- [`1.16.1`, (*1.16.1/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.16.1)
- [`1.16.0`, (*1.16.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.16.0)
- [`1.15.0`, (*1.15.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.15.0)
- [`1.14.0`, (*1.14.0/Dockerfile*)](https://github.com/MatthewVance/unbound-docker-rpi/tree/master/1.14.0)

## What is Unbound?

Unbound is a validating, recursive, and caching DNS resolver.
> [unbound.net](https://unbound.net/)

## How to use this image

### Standard usage

Pull this container:
`docker pull mvance/unbound-rpi:latest`

Run this container with the following command:

```console
docker run \
--name=unbound-rpi \
--publish=53:53/udp \
--publish=53:53/tcp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

### Override default forward

By default, forwarders are configured to use Cloudflare DNS. You can retrieve the configuration in the [1.13.1/forward-records.conf](1.13.1/forward-records.conf) file.

You can create your own configuration file and override the one placed in `/opt/unbound/etc/unbound/forward-records.conf` in the container.

Example `forward-records.conf`:
```
forward-zone:
  # Forward all queries (except those in cache and local zone) to
  # upstream recursive servers
  name: "."

  # my DNS
  forward-addr: 192.168.0.1@53#home.local
```

Once the file has your entries in it, mount your version of the file as a volume
when starting the container:

```console
docker run \
--name=my-unbound \
--volume=$(pwd)/forward-records.conf:/opt/unbound/etc/unbound/forward-records.conf:ro \
--publish=53:53/udp \
--publish=53:53/tcp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

### Serve Custom DNS Records for Local Network

While Unbound is not a full authoritative name server, it supports resolving
custom entries on a small, private LAN. In other words, you can use Unbound to
resolve fake names such as your-computer.local within your LAN.

To support such custom entries using this image, you need to provide an
`a-records.conf` file. This conf file is where you will define your custom
entries for forward and reverse resolution.

The `a-records.conf` file should use the following format:

```
# A Record
  #local-data: "somecomputer.local. A 192.168.1.1"
  local-data: "laptop.local. A 192.168.1.2"

# PTR Record
  #local-data-ptr: "192.168.1.1 somecomputer.local."
  local-data-ptr: "192.168.1.2 laptop.local."
```

Once the file has your entries in it, mount your version of the file as a volume
when starting the container:

```console
docker run \
--name=unbound-rpi \
--volume=$(pwd)/a-records.conf:/opt/unbound/etc/unbound/a-records.conf:ro \
--publish=53:53/udp \
--publish=53:53/tcp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

#### SRV records

The `srv-records.conf` file should use the following format:

```
# SRV records
# _service._proto.name. | TTL | class | SRV | priority | weight | port | target.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-0.domain.local.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-1.domain.local.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-2.domain.local.
```

Run a container that use this SRV config file:
```console
docker run \
--name my-unbound \
--volume=$(pwd)/srv-records.conf:/opt/unbound/etc/unbound/srv-records.conf:ro \
--publish=53:53/udp \
--publish=53:53/tcp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

### Append customizations to the provided Unbound configuration

If you would like to add more complex custom configurations while maintaining
the defaults generated by the unbound.sh script, use the wildcard include 
(feature added in the 1.13.0 image):

```console
docker run --name=unbound-rpi \
--volume=/my-directory/unbound:/opt/unbound/etc/unbound/ \
--publish=53:53/tcp \
--publish=53:53/udp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

*Note: See https://github.com/MatthewVance/unbound-docker-rpi/issues/8 for more background.*

### Use a fully customize Unbound configuration

Instead of using this image's default configuration for Unbound, you may supply
your own configuration. If your customized configuration is located at 
`/my-directory/unbound/unbound.conf`, pass `/my-directory/unbound` as a volume
when creating your container:

```console
docker run --name=unbound-rpi \
--volume=/my-directory/unbound:/opt/unbound/etc/unbound/ \
--publish=53:53/tcp \
--publish=53:53/udp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest
```

This will expose all files in `/my-directory/unbound/` to the container. As an
alternate way to serve custom DNS records for any local zones, either place 
them directly in your `unbound.conf`, or place the local zones in a separate
file and use Unbound's include directive within your `unbound.conf`:

```
include: /opt/unbound/etc/unbound/local-zone-unbound.conf
```

Your volume's contents might eventually look something like this:

```
/my-directory/unbound/
-- unbound.conf
-- local-zone-unbound.conf
-- secret-zone.conf
-- some-other.conf
```

Overall, this approach is very similar to the `a-records.conf` approach described above. However, passing your unbound directory gives you more options for customizing and segmenting your Unbound configuration than a single file does.

***Note:** Care has been taken in the default configuration to enable
security options so it is recommended to use it as a guide.*

# User feedback

## Documentation

Documentation for this image is stored right here in the [`README.md`](https://github.com/MatthewVance/unbound-docker-rpi/blob/master/README.md).

Documentation for Unbound is available on the [project's website](https://unbound.net/).

# Notes

## Logging

Logging is very limited in the default config created by unbound.sh. If using the default config as an example starting point, a placeholder for a logfile (`unbound.log`) has been provided with the correct file ownership at the path `/opt/unbound/etc/unbound/` in case you want to increase logging and send to a file.

## Issues

If you have any problems with or questions about this image, please contact me
through a [GitHub issue](https://github.com/MatthewVance/unbound-docker-rpi/issues).

*Note: IPv6 is not supported for this image.*

## Contributing

You are invited to contribute new features, fixes, or updates, large or small. I
imagine the upstream projects would be equally pleased to receive your
contributions.

Please familiarize yourself with the [repository's `README.md`
file](https://github.com/MatthewVance/unbound-docker-rpi/blob/master/README.md)
before attempting a pull request.

Before you start to code, I recommend discussing your plans through a [GitHub
issue](https://github.com/MatthewVance/unbound-docker-rpi/issues), especially for
more ambitious contributions. This gives other contributors a chance to point
you in the right direction, give you feedback on your design, and help you find
out if someone else is working on the same thing.

## Acknowledgments

The code in this image is heavily influenced by DNSCrypt server Docker image,
though the upstream projects most certainly also deserve credit for making this
all possible.
- [Docker](https://www.docker.com/)
- [DNSCrypt server Docker image](https://github.com/jedisct1/dnscrypt-server-docker)
- [OpenSSL](https://www.openssl.org/)
- [Unbound](https://unbound.nlnetlabs.nl/)

## Licenses

### License

Unless otherwise specified, all code is released under the MIT License (MIT).
See the [repository's `LICENSE`
file](https://github.com/MatthewVance/unbound-docker-rpi/blob/master/LICENSE) for
details.

### Licenses for other components

- Docker: [Apache 2.0](https://github.com/docker/docker/blob/master/LICENSE)
- DNSCrypt server Docker image: [ISC License](https://github.com/jedisct1/dnscrypt-server-docker/blob/master/LICENSE)
- OpenSSL: [Apache-style license](https://www.openssl.org/source/license.html)
- Unbound: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)
