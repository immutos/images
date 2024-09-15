# images

Pipeline for building immutos Debian images.


## Debian Bookworm

### Debian Bookworm UltraSlim

A very minimal Debian Bookworm image intended to be used as a base image for
containers.

* **Reference**: `registry.dpeckett.dev/debian:bookworm-ultraslim`
* **Architectures**: amd64, arm64

### Debian Bookworm UltraSlim Bootable

A very minimal bootable Debian Bookworm image intended to be used as a base
file system for bootable images.

* **Reference**: `registry.dpeckett.dev/debian:bookworm-ultraslim-bootable`
* **Architectures**: amd64, arm64

## Debian Trixie (Testing)

Testing images for the next Debian release. The main reason for publishing these
is that Trixie will be the first release to support the riscv64 architecture.

### Debian Trixie UltraSlim

A very minimal Debian testing image intended to be used as a base image for
containers.

* **Reference**: `registry.dpeckett.dev/debian:trixie-ultraslim`
* **Architectures**: amd64, arm64, riscv64

### Debian Trixie UltraSlim Bootable

A very minimal bootable Debian testing image intended to be used as a base
file system for bootable images.

* **Reference**: `registry.dpeckett.dev/debian:trixie-ultraslim-bootable`
* **Architectures**: amd64, arm64, riscv64

## Usage

### Runnning bootable images

To run a bootable image, without a virtual machine, you can use the following:

```shell
docker run --rm -it \
  --cap-add SYS_ADMIN \
  --security-opt apparmor=unconfined \
  --security-opt seccomp=unconfined \
  --tmpfs /run --tmpfs /run/lock \
  --cgroupns=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup \
  registry.dpeckett.dev/debian:bookworm-ultraslim-bootable
```