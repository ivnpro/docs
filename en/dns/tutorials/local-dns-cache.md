# Configuring a local caching DNS resolver

**Don't** use {{ dns-full-name }} directly for requesting external domain names.

Instead, install and configure a local caching DNS resolver like `systemd-resolved`,
`dnsmasq`, or `unbound`. This will speed up the execution of public DNS queries, reduce their number, and [save you money](../pricing.md#public-dns-requests).

If Ubuntu 18.04 LTS or higher is installed on your VM, it already has the `systemd-resolved` service by default and no additional installation and setup is required. For more information, see [systemd-resolved](#systemd-resolved).

If you're using an older version of Ubuntu, such as 16.04 LTS, you need to install a caching DNS resolver yourself. We recommend using `dnsmasq`
or `unbound`. For more information, see [Alternative resolvers](#alternate-resolvers).

## systemd-resolved in Ubuntu 18.04+ {#systemd-resolved}

### Health check {#test-resolver}

You can get the status of the `systemd-resolved` caching DNS resolver in Ubuntu 18.04+ by running the command:

```
systemd-resolve --status
```

Command output:

```
<...>
Link 2 (eth0)
  <...>
  Current DNS Server: 192.168.6.2         
         DNS Servers: 192.168.6.2         
          DNS Domain: ru-central1.internal
                      auto.internal 
```

If the configuration is correct, the `Current DNS Server:` line in the command output will show the {{ dns-full-name }} server address that is the second address in the virtual network. For example, `192.168.6.2` for a VM

You can use the `dig` utility to make sure that external DNS names are resolved by the `127.0.0.53#53` DNS server.
Any resolve of an external DNS name, except for the very first one, is executed almost instantly:

```
dig example.com
<...>
;; Query time: 69 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
<...>
   
dig example.com
<...>
;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
<...>
```

### Using an alternative resolver {#change-resolver}

If you need to replace `systemd-resolved` with a different local caching DNS resolver:

1. Follow the instructions in [Alternative resolvers](#alternate-resolvers) to set an alternative resolver.

2. Edit the `/etc/systemd/resolved.conf` file to disable a stub resolver

   ```
   [Resolve]
   <...>
   DNS=127.0.0.1
   DNSStubListener=no
   ```

3. Restart `systemd-resolved`:

   ```
   systemctl restart systemd-resolved
   ```

4. Check that `dig` now uses the `127.0.0.1#53` server instead of `127.0.0.53#53`:

   ```
   dig example.com
   ```

   Result:

   ```
   <...>
   ;; Query time: 0 msec
   ;; SERVER: 127.0.0.1#53(127.0.0.1)
   <...>
   ```

## Alternative resolvers {#alternate-resolvers}

{% note warning %}

Configuration parameters are given as an example. For information about how to set up a caching resolver based on the VM's workload and your needs, see the documentation for the selected resolver.

{% endnote %}

{% list tabs %}

- dnsmasq

  [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) is lightweight caching DNS resolver that has a small footprint.

  1. Install the `dnsmasq` and `dnsutils` packages:

     ```
     sudo apt-get update -y
     
     sudo apt-get install dnsmasq dnsutils -y
     ```

  2. Edit the `/etc/dnsmasq.conf` file by increasing the cache size to 1000

     ```
     cache-size=1000
     listen-address=127.0.0.1
     interface=lo
     bind-interfaces
     ```

  3. Restart `dnsmasq` to apply the new settings. Check that the `dnsmasq` service status changed to `active (running)` and there is a line saying `Started dnsmasq.service` at the end of the service log:

     ```
     sudo systemctl restart dnsmasq.service
     
     sudo systemctl status dnsmasq.service
     ● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/lib/systemd/system/dnsmasq.service; enabled; vendor preset: enabled)
     Active: active (running) <...>
     <...>
     Oct 28 22:39:57 <...> systemd[1]: Starting dnsmasq - A lightweight DHCP and caching DNS server...
     Oct 28 22:39:57 <...> dnsmasq[<...>]: dnsmasq: syntax check OK.
     Oct 28 22:39:57 <...> dnsmasq[<...>]: started, version 2.75 cachesize 150
     <...>
     Oct 28 22:39:58 <...> dnsmasq[<...>]: using nameserver 192.168.6.2#53
     Oct 28 22:39:58 <...> systemd[1]: Started dnsmasq - A lightweight DHCP and caching DNS server.
     ```

     Also pay attention to the address in the `using nameserver <...>` line. It should specify the address of the {{ dns-full-name }} server that is the second address in the virtual network. For example, `192.168.6.2` for a VM

  5. Use `dig` to check that external DNS names are resolved successfully:

     ```
     $ dig example.com @127.0.0.1 | grep -B3 Query
     ;; ANSWER SECTION:
     example.com.		65420	IN	A	93.184.216.34
     ;; Query time: 3 msec
     
     $ dig example.com @127.0.0.1 | grep -B3 Query
     ;; ANSWER SECTION:
     example.com.		65420	IN	A	93.184.216.34
     ;; Query time: 0 msec
     ```

- unbound

  [unbound](https://unbound.docs.nlnetlabs.nl/en/latest/) is a validating, recursive, and caching DNS resolver. It focuses on fast performance, efficient resource consumption, and compliance with modern open standards.

  1. Install the `unbound` and `dnsutils` packages:

     ```
     sudo apt-get update -y
     
     sudo apt-get install unbound dnsutils -y
     ```

  2. Specify the server settings at the end of the global configuration file `/etc/unbound/unbound.conf`
     or in a separate configuration file named `/etc/unbound/unbound.conf.d/<file name>.conf`:

     ```
     server:
         port: 53
         interface: 127.0.0.1
         access-control: 127.0.0.0/8 allow
         do-ip4: yes
         do-ip6: no
         do-udp: yes
         do-tcp: yes
         num-threads: 2
         num-queries-per-thread: 1024
         hide-identity: yes
         hide-version: yes
         prefetch: yes
         verbosity: 1
     ```

  3. Start `unbound` to apply the new settings. Check that the `unbound` service status changed to `active (running)` and there is a line saying `Started unbound.service` at the end of the service log:

     ```
     sudo systemctl start unbound.service
     
     sudo systemctl status unbound.service
     ● unbound.service - Unbound DNS server
     Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor preset: enabled)
     Active: active (running) <...>
     <...>
     Oct 29 00:21:06 <...> unbound[<...>]:  * Starting DNS server unbound
     Oct 29 00:21:06 <...> unbound[<...>]: [<...>:0] info: start of service (unbound 1.5.8).
     Oct 29 00:21:06 <...> unbound[<...>]:    ...done.
     Oct 29 00:23:21 <...> systemd[1]: Started unbound.service.
     ```

  4. Use `dig` to check that external DNS names are resolved successfully:

     ```
     dig example.com @127.0.0.1 | grep -B3 Query
     ;; ANSWER SECTION:
     example.com.		86400	IN	A	93.184.216.34
     ;; Query time: 611 msec
     
     dig example.com @127.0.0.1 | grep -B3 Query
     ;; ANSWER SECTION:
     example.com.		86397	IN	A	93.184.216.34
     ;; Query time: 0 msec
     ```

{% endlist %}
