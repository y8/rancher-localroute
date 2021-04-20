⛔️ This fix it not relevant anymore, repository kept for historical purposes 

# Rancher local routing fix

[![](https://badge.imagelayers.io/yopp/rancher-localroute:latest.svg)](https://imagelayers.io/?images=yopp/rancher-localroute:latest 'Get your own badge on imagelayers.io')

Problem: if you have Rancher service that exposed port to public IP, you can't
access that port from the host network. For example, you have two servers
`server0` and `server1`. Load Balancer is deployed on `server0` to provide
Docker Registry on `hub.example.net:443` and `hub.example.net` resolves to `server0`
public IP.

Due Ranched Managed Network design, you can't access `hub.example.net:443` on
the `server0`. It will be accessible from the `server1` and the world thought.

You need to enable IP forwarding and local routing on all interfaces. To be
sure to be sure, just add

````
net.ipv4.ip_forward=1
net.ipv4.conf.all.route_localnet=1
````

To `/etc/sysctl.d/11-rancher.conf` file and reboot the host. Afterwards you
can deploy the container.

This solution is verified on `Ubuntu 14.04` with `3.19.0-33-generic` kernel.

See: <https://github.com/rancher/rancher/issues/147> and
<https://github.com/rancher/rancher/issues/2929> for details.

Note: containers **must** be started with `--privileged --net="host"` options in
order to update `sysctl` settings and read and `iptables` chains on the
host system.

You can deploy this container with Rancher:

  * Add new service on any stack
  * Click "Advanced Options"
  * Change "Scale" to "Always run one instance of this container on every host"
  * Open "Networking" tab and set "Network" to "Host
  * Open "Security/Host" and check the "Full access to the host"

Repeat for all environments.

There two configuration environment variables available:

  * `POLLER_HOST_IP` to explicitly set the host IP addresses. You can set
     multiple addresses by separating them with comma, like `10.0.1.1, 10.0.1.2`
  * `POLLER_IFACE` to set the interface name, like `eth0` where IP addresses
     will be auto-detected. If new IP address is added or removed from the
     given interface routing chain will change as well. Addresses with
     `IFF_LOOPBACK` flag will be ignored.

If no environment variables set first interface with non loopback IPv4 will be
used.

`rancher-localroute` works only with IPv4 and doesn't support IPv6 at all.

This is very quick and dirty fix, so keep an eye on the logs. And check for
the rancher-server issues for updates.
