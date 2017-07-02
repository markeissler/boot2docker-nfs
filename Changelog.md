# Changelog: boot2docker-nfs

## 0.10.0 / 2017-07-01

Log rotation has been added for all system logs, no need to worry about long-running containers.

### Short list of commit messages

  * Update kernel to 4.4.74.
  * Update AUFS to v4.4-20170612.
  * Build, install, configure logrotate.
  * Updated os-release urls.

## 0.9.0 / 2017-06-10

Initial release! Adds `Dockerfile.nfs` to generate an ISO that supports NFS
client services on boot.

### Short list of commit messages

  * Update README for v0.9.0.
  * Update docs for NFS support.
  * Add support for building b2d with NFSv3 client support at startup.
