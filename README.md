# GRUB enhancements for ZFS on Linux

The home for this git repository is:

* https://github.com/zfsonlinux/grub

GRUB packages at https://launchpad.net/~zfs-native/+archive/grub
(the PPA) and http://archive.zfsonlinux.org/debian/ are built
from this repository using the git-buildpackage tool.


## Casual Build Instructions

If you are using APT to install Grub, then just do quick builds like this:
```
$ apt-get source --build grub
```
This requires a corresponding `deb-src` line for each `deb` line in the
`/etc/apt/sources.list.d` file for Grub.


## Developer Build Instructions

1. Clone this repository:
   ```
   $ git clone git://github.com/zfsonlinux/grub.git
   $ cd grub
   ```

2. List the current releases by branch name:
   ```
   $ git branch --list 'master/*'
   ```

3. Or list previous releases by tag name:
   ```
   $ git tag --list 'master/*'
   $ git tag --list 'snapshot/*'
   ```

4. Before building the chosen tag/branch, because of limitations and faults
with the current ZoL development package, you need to modify the includes.
This is done using the command
   ```
   $ ./fix_includes-libspl.sh
   ```
This is, however, no longer need if using the 0.6.3-41-0f7d2a version or
newer. It's now done when packaging that.

5. Checkout the branch name or tag name that you want to build.  For example,
the latest code for Ubuntu 12.04 Raring is:
   ```
   $ git checkout master/ubuntu/raring/2.00-13ubuntu3+zfs3_raring
   ```

6. Now compile it:
   ```
   $ git-buildpackage -uc -us
   ```

7. And clean the working tree afterwards by doing this:
   ```
   $ git clean -df
   $ git reset --hard
   ```

# Release Instructions

1. Build a binary+source release like this:
   ```
   $ git-buildpackage --git-tag [-sa|-sd]
   ```
(The `-sa` switch means "upload a new upstream tarball" for an out-series
build. The `-sd` switch means "only upload the new overlay" for an in-series
builds.)

2. Synchronize the release bucket to your working copy.
   ```
   $ s3cmd sync --dry-run s3://archive.zfsonlinux.org/ ./archive.zfsonlinux.org/
   $ s3cmd sync s3://archive.zfsonlinux.org/ ./archive.zfsonlinux.org/
   ```

3. Update the release bucket like this:
   ```
   $ cd ./archive/zfsonlinux.org/debian/
   $ reprepro include wheezy /tmp/zfs-linux_${version}_amd64.changes
   ```

4. Do a local installation in a clean sandbox to ensure that the Release and
Sources are sensible.

5. Give notice that you're touching the release bucket, and synchronize the new
packages:
   ```
   $ s3cmd sync --dry-run ./archive.zfsonlinux.org/ s3://archive.zfsonlinux.org/
   $ s3cmd sync ./archive.zfsonlinux.org/ s3://archive.zfsonlinux.org/
   ```

(Ideally, you would sync the pool first, and then sync the meta to ensure the
smallest possible window of inconsistency.)


## Upstream Repositories

The `upstream` branch in this repository is an unmodified copy of the
http://github.com/zfsonlinux/grub.git mainline.
