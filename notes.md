All node sin NOT READY => CNI plugin not working

## crictl

- Very much like docker
- used as a lowlevel component, that kubectl or kubelet uses.
    - if kubectl / kube-api doesn't work, this is the cli we reach for

```bash
sudo crictl ps -a                    # list containers, including crashed ones
sudo crictl logs <container-id>      # the actual error message
```
