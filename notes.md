All node sin NOT READY => CNI plugin not working

## crictl

- Very much like docker
- used as a lowlevel component, that kubectl or kubelet uses.
    - if kubectl / kube-api doesn't work, this is the cli we reach for

```bash
sudo crictl ps -a                    # list containers, including crashed ones
sudo crictl logs <container-id>      # the actual error message
```


## etc/hosts | ip routes | NAT

/etc/hosts => rules for (DNS_NAME) => IP

NAT => rewrites IPs to IPs, (IP) => IP

IP Routes => where to send this package to? (IP) => what "carries" this packat?

the carrier might be a userspace application, VPNs and Proxies work that way
