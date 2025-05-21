# Tips for debugging



## Pod creation fails with "error unpacking image"

Sometimes when creating a pod you might encounter the following error:

```sh
Warning  Failed     8m51s (x7 over 10m)  kubelet            Error: failed to create containerd container: error unpacking image: failed to extract layer sha256:d51af96cf93e225825efd484ea457f867cb2b967f7415b9a3b7e65a2f803838a: failed to get reader from content store: content digest sha256:ec562eabd705d25bfea8c8d79e4610775e375524af00552fe871d3338261563c: not found
```

If you encounter this error, first check if the `discard_unpacked_layers`
setting is enabled in the containerd configuration. This setting removes
compressed image layers after unpacking, which can cause issues because
Confidential Containers Operator workloads rely on those layers. To disable it, update
`/etc/containerd/config.toml` with:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd]
  discard_unpacked_layers = false
```


>ℹ️ Previously failed images needs to be prefetched
```sh
ctr -n k8s.io content fetch <image>
```
Restart containerd after making the change:

```bash
sudo systemctl restart containerd
```
>ℹ️ If on a remote, it will kill the connection.



## Debug container

It can be quite usefull to start a debug container, usually kubernetes nodes are stripped / hardened, which means no tools are installed, a debug container can be any image.

`Debug container`
```sh
kubectl debug node/dev01-worker -it --image=busybox
```

When in a debug container host system are mounted under */host*

`Edit containerd config`
```sh
vi /host/etc/containerd/config.toml
```

`Show containerd log`
```sh
chroot /host
journalctl -xeu containerd
```

`Edit kata config`
```sh
vi /host/opt/kata/containerd/config.d/kata-deploy.toml
```

`Edit qemu-coco-dev config`
```sh
vi /host/opt/kata/share/defaults/kata-containers/configuration-qemu-coco-dev.toml
```

[General troubleshooting guide](https://github.com/confidential-containers/confidential-containers/blob/main/guides/troubleshooting.md).

