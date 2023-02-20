As [pointed out](https://github.com/mochman/Bypass_CGNAT/issues/33) by [@Leolele99](https://github.com/Leolele99):

_When installing with default values, the wireguard VPN initializes its wg0 interface with a MTU of 1420._
_While this is fine for most network interfaces (Modern standard MTU size seems to be 1500), this poses a peculiar problem with docker and docker compose:_

_The docker daemon (and therefore default bridge network and docker0 network interface) MTU is 1500. All services running directly on the host or containers using the host network, will accept the new MTU of 1420 imposed by wireguard/wg0, but everything using the default or custom docker bridge networks, will use the docker default MTU of 1500._
_When making outside requests from within such a container, the packets need to be split to fit the smaller MTU. Theoretically this would just result in more inefficient traffic, but practically, many networks, firewalls, or cloud services prohibit these fragmented packages from passing through, resulting in lost packets, dropped connections, or timeouts._
_This is very hard to debug and can be entirely sporadic._

_Additionally, some cards/adapters have a smaller MTU than 1500, which could result in the same package loss._

If you are experiencing issues with your docker services through the tunnel, you can try following the steps outlined on [https://www.civo.com/learn/fixing-networking-for-docker](https://www.civo.com/learn/fixing-networking-for-docker)