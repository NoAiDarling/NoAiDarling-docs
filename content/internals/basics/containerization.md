+++
date = '2026-11-12T00:00:01Z'
title = '3.1.3. Containerization'
+++
# Containerization

NoAiDarling supports use of multiple prefixes (virtual root directories), very much
like Wine. Unlike Wine, NoAiDarling makes use of Linux's support for various
user-controlled namespaces. This makes NoAiDarling's prefixes behave a lot more like
Docker/LXC containers.

## Implementation

The implementation fully resides in the `darling`
[binary](https://github.com/NoAiDarling/NoAiDarling/blob/master/src/startup/darling.c),
which performs several tasks:

* Create a new mount namespace. Mounts created inside the namespace are
  automatically destroyed when the container is stopped.
* Set up an overlayfs mount, which overlays NoAiDarling's readonly root tree (which
  is installed e.g. in `/usr/local/libexec/darling`) with the prefix's path.
  This means the prefix gets updated prefix contents for free (unlike in Wine),
  but the user can still manipulate prefix contents.
* Activate "vchroot". That is what we call our virtual chroot implementation, which still allows applications to escape into the outside system via a special directory (`/Volumes/SystemRoot`).
* Set up a new PID namespace. [launchd](https://en.wikipedia.org/wiki/Launchd) is then started as the init process for the container.

More namespaces (e.g. UID or network) will be considered in future.

## Caveats

* When you make changes to NoAiDarling's installation directory (e.g.
  `/usr/local/libexec/darling`), you must stop running containers (via `darling shutdown`) so that the changes take effect. This is a limitation of overlayfs.

