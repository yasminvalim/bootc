
# Container runtime vs "bootc runtime"

Fundamentally, `bootc` reuses the [OCI image format](https://github.com/opencontainers/image-spec)
as a way to transport serialized filesystem trees with included metadata such as a `version`
label, etc.

However, `bootc` generally ignores the [Container configuration](https://github.com/opencontainers/image-spec/blob/main/config.md)
section at runtime today.

Container runtimes like `podman` and `docker` of course *will* interpret this metadata
when running a bootc container image as a container.

## Labels

A key aspect of OCI is the ability to use standardized (or semi-standardized)
labels.  The are stored and rendered by `bootc`; especially the
`org.opencontainers.image.version` label.

## Example ignored runtime metadata, and recommendations

### `ENTRYPOINT` and `CMD` (OCI: `Entrypoint`/`Cmd`)

Ignored by bootc.

It's recommended for bootc containers to set `CMD /sbin/init`; but this is not required.

The booted host system will launch from the bootloader, to the kernel+initramfs and
real root however it is "physically" configured inside the image.  Typically
today this is using [systemd](https://systemd.io/) in both the initramfs
and at runtime; but this is up to how you build the image.

### `ENV` (OCI: `Env`)

Ignored by bootc; to configure the global system environment you can
change the systemd configuration.  (Though this is generally not a good idea;
instead it's usually better to change the environment of individual services)

### `EXPOSE` (OCI: `exposedPorts`)

Ignored by bootc; it is agnostic to how the system firewall and network
function at runtime.

### `USER` (OCI: `User`)

Ignored by bootc; typically you should configure individual services inside
the bootc container to run as unprivileged users instead.

### `HEALTHCHECK` (OCI: *no equivalent*)

This is currently a Docker-specific metadata, and did not make it into the
OCI standards.  (Note [podman healthchecks](https://developers.redhat.com/blog/2019/04/18/monitoring-container-vitality-and-availability-with-podman#))

It is important to understand again is that there is no "outer container runtime" when a
bootc container is deployed on a host.  The system must perform health checking on itself (or have an external
system do it).

Relevant links:

- [bootc rollback](../man/bootc-rollback.md)
- [CentOS Automotive SIG unattended updates](https://sigs.centos.org/automotive/building/unattended_updates/#watchdog-in-qemu)
  (note that as of right now, greenboot does not yet integrate with bootc)
- <https://systemd.io/AUTOMATIC_BOOT_ASSESSMENT/>
