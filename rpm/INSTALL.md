# Building RPMs #

All of the code to build the RPM is stored in a SPECS/git-lfs.spec file. The
source code tarball needs to be put in a SOURCES directory. The BUILD and
BUILDROOT directories are used during the build process. The final RPM ends up
in the RPMS directory and a source-rpm in SRPMS.

In order to expedite installing all dependencies (mainly ruby-ronn and golang)
and download any needed files a build_rpms.bsh script is included. This is the
**RECOMMENDED** way to build the rpms. It will install all yum packages in
order to build the rpm. This can be especially difficult in CentOS 5 and 6,
but it will build and install a suitable golang/ruby so that git-lfs can be
built.

Simple run:

```
./clean.bsh
./build_rpms.bsh
```

The clean.bsh script removes previous rpms, etc... and removed the source
tar.gz file. Otherwise you might end up creating an rpm with pieces from
different versions.

Practice is to run rpmbuild as non-root user. This prevents inadvertently
installing files in the operating system. The intent is to run build_rpms.bsh
as a non-root user with sudo privileges. If you have a different command for
sudo, or do not have sudo installed (which is possible, but unlikely), you can
set the SUDO environment variable to nothing or another command and you can
run as root if that is your style. Example:

```
./clean.bsh
SUDO=echo ./build_rpms.bsh
  or
(as root) SUDO= ./build_rpms.bsh
```

(The echo example will let you know what yum commands you need to run to make
the build work in case you care. Most of people will just run
```./build_rpms.bsh``` and be done.)

When all is down, install (or distribute) RPMS/git-lfs.rpm

```
yum install RPMS/x86_64/git-lfs*.rpm
```

### Alternative build method ###

If you want to use your own ruby/golang without using the version from
build_rpms.bsh, you will have to disable dependencies on the rpms. It's pretty
easy, just make sure ronn and go are in the path, and run

```
NODEPS=1 ./build_rpms.bsh
```

### Manual build method ###

If you want to use your own ruby/golang without using build_rpms.bsh, it's a
little more complicated. You have to make sure ronn and go are in the path,
and create the build structure, and download/create the tar.gz file used. This
is not recommended, but it is possible.

```
mkdir -p {BUILD,BUILDROOT,SOURCES,RPMS,SRPMS}
#download file to SOURCES/v{version number}.tar.gz
rpmbuild --define "_topdir `pwd`" -bb SPECS/git-lfs.spec --nodeps

#(and optionally)
rpmbuild --define "_topdir `pwd`" -bs SPECS/git-lfs.spec --nodeps
```

### Releases ###

It is no longer necessary to update SPECS/git-lfs.spec every version. As long
as lfs/lfs.go is updated, build_rpms.bsh parses the version number using the
pattern ```s|const Version = "\([0-9.]*\)"|\1|``` and updates
SPECS/git-lfs.spec. The version number is then used to download:

https://github.com/github/git-lfs/archive/v%{version}.tar.gz

This way when a new version is archived, it will get downloaded and built
against. When developing, it is advantageous to use the currently checked out
version to test against. In order do that, after running ```./clean.bsh```,
set the environment variable BUILD_LOCAL to 1

```
./clean.bsh
BUILD_LOCAL=1 ./build_rpms.bsh
```

### Troubleshooting ###

**Q**) "error: Bad owner/group" when building SRPM (rpmbuild -bs command)

**A**) For some... STUPID reason, git-lfs.spec has to be OWNED by a valid user.
Just chown git-lfs.spec to a valid user AND group.
```chown root:root git-lfs.spec``` will do.
