# Configuring Your Firewall

!!! info ""
    You are welcome to ask for help in our [community chat](https://link.photoprism.app/chat).
    [Sponsors](https://www.photoprism.app/membership) receive direct [technical support](https://www.photoprism.app/contact) via email.
    Before [submitting a support request](../../user-guide/index.md#getting-support), try to [determine the cause of your problem](index.md).

## Incoming Requests

Unless you have changed the default configuration, PhotoPrism is reachable via port 2342 on all network devices. If you are using a firewall, please ensure that this port can be accessed from other computers on your network, or that your instance can be accessed [through a reverse proxy](../proxies/traefik.md):

![](https://dl.photoprism.app/img/diagrams/proxy-cdn.svg){ class="w100" }

## Outgoing Connections

As explained in our [Privacy Policy](https://www.photoprism.app/privacy#section-7), reverse geocoding and interactive world maps depend on retrieving the necessary information [from us](https://www.photoprism.app/contact) and [MapTiler AG](https://www.maptiler.com/contacts/), headquartered in Switzerland. Both services are provided with a very high level of privacy and confidentiality.

[View Privacy Policy ›](https://www.photoprism.app/privacy#section-7){ class="pr-3 block-xs" } [View Compliance FAQ ›](https://www.photoprism.app/kb/compliance-faq#privacy)

In order to successfully set up your installation and view location details in PhotoPrism, you must **allow requests to the following hosts** if you have a firewall installed, and make sure that your Internet connection is working:

- [ ] [dl.photoprism.app](https://dl.photoprism.app/ "File Downloads")
- [ ] [my.photoprism.app](https://my.photoprism.app/ "Member Portal")
- [ ] [cdn.photoprism.app](https://cdn.photoprism.app/ "Content Delivery Network (CDN)")
- [ ] [maps.photoprism.app](https://maps.photoprism.app/ "Vector Map Tiles")
- [ ] [places.photoprism.app](https://places.photoprism.app/ "Reverse Geocoding API")
- [ ] [places.photoprism.xyz](https://places.photoprism.xyz/ "Reverse Geocoding API")

In addition, the following API endpoints should be allowed so that public [Docker](https://www.docker.com/) images can be pulled from [Docker Hub](https://hub.docker.com/):

- [ ] auth.docker.io
- [ ] registry-1.docker.io
- [ ] index.docker.io
- [ ] dseasb33srnrn.cloudfront.net
- [ ] production.cloudflare.docker.com

## IPTables and Docker

On Linux, Docker manipulates the `iptables` rules to provide network isolation. This does have some implications for what you need to do if you want to have your own policies in addition to the rules Docker manages.

[Learn more ›](https://docs.docker.com/network/iptables/)

## Docker MTU Size

If you use Docker on your server or on a virtual machine, technical limitations of the local network or your internet provider can also make it impossible to reach external services. In particular, the *network cards of virtual machines* often do not have the standard [Maximum Transmission Unit (MTU)](https://en.wikipedia.org/wiki/Maximum_transmission_unit) of 1500, but a smaller size like 1492 or 1454.

In this case, you must [configure the virtual network cards](https://mlohr.com/docker-mtu/) of your Docker containers so that they have an MTU size that is less than or equal to that of the outgoing network, for example by [adding the following](https://www.civo.com/learn/fixing-networking-for-docker) to your `compose.yaml` (or `docker-compose.yml`) config files:

```yaml
networks:
  default:
    driver: bridge
    driver_opts:
      com.docker.network.driver.mtu: 1450
```

[Learn more ›](https://mlohr.com/docker-mtu/)

!!! note ""
    All network configuration changes require a restart of the affected services and/or the Docker daemon to take effect.
