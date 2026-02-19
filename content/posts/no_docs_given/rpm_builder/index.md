+++
title = "Why Redis Doesn't Ship an RPM (and How to Build One)"
date = "2026-02-18T18:38:04-05:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = ""
authorTwitter = "" #do not include @
cover = "drake_meme.png"
tags = ["rpm", "redis", "linux", "packaging", "rhel", "rpmbuild"]
keywords = ["rpm", "redis", "yum", "dnf", "spec file", "rpmbuild", "rhel", "rocky linux", "amazon linux"]
description = "Why Redis offers an official APT repo but not a YUM repo, plus a practical guide to RPM packaging and building with spec files on RHEL-based systems."
showFullContent = false
readingTime = true
hideComments = false
+++

<what is an rpm>

Stop me if this sounds familiar: you want to run a popular service on your cloud instance. You go to the vendor site and look for the easiest way to install something on a DNF-based cloud Linux server. You look for the RPM for your Linux distro. Wait, it is not there? How can that be? They've got RPMs for everything else.

I want to install the latest 7 version. The latest stable Redis 7 version is 7.2.4, released in January 2024.

Let's take a real world example with Redis. These are their install directions for Linux: https://redis.io/docs/latest/operate/oss_and_stack/install/archive/install-redis/install-redis-on-linux/


## Why Do I See an Official APT Repo… But Not an RPM Repo?

If you go to the Redis install page, you’ll notice something interesting.

On Debian/Ubuntu you’re told to add an official upstream APT repository:

```bash
echo "deb https://packages.redis.io/deb ..."

```

But on Red Hat–based systems (RHEL, Rocky, Amazon Linux), you’re told:

```bash
sudo yum install redis
```
No official redis.io YUM repo. No GPG key setup. No custom repo file.

### Redis runs perfectly fine on RPM-based systems.

The difference isn’t technical — it’s ecosystem philosophy.

In Debian-based systems:

* Adding upstream vendor repositories is common.
* Users frequently add repos for Docker, Node.js, PostgreSQL, etc.
* Faster upstream release cadence is expected.
* Version freshness is prioritized.

So Redis maintains an official APT repository to control versioning and updates directly.

RPM-based enterprise systems work differently.

The expectation is:
* The distribution packages the software.
* Stability matters more than latest features.
* Backported security patches are preferred over major upgrades.
* Enterprises avoid random upstream repos when possible.

So Redis does not maintain an official YUM repo in the same way.

Instead, Redis is packaged by the distribution maintainers.

---

{{< figure src="do_something.png"  style="max-width:70%;" >}}

## RPMs and what they really do

An RPM (Red Hat Package Manager) is a structured software package format used in RHEL-based Linux systems that installs, configures, tracks, verifies, and cleanly removes software — including its files, dependencies, and lifecycle scripts — in a consistent and reproducible way.

For the longest time I thought it was the most magical thing — it would somehow create the binary, the systemd service file, the logrotate config, the users, the directories… like Linux just knew what to do.

But here’s the truth:

An RPM doesn’t magically create anything.

It packages and orchestrates everything you define.

More specifically, many RPMs include:

* The compiled binary (e.g. /usr/bin/redis-server)
* A systemd service file (/usr/lib/systemd/system/redis.service)
* A logrotate configuration (/etc/logrotate.d/redis)
* Configuration files (/etc/redis/redis.conf)
* Default directory structures (/var/lib/redis, /var/log/redis)
* User and group creation (via %pre scripts)
* Post-install logic (via %post)
* Pre-uninstall cleanup (via %preun)
* File ownership and permissions
* Dependency metadata
* Version tracking and upgrade logic

It’s not magic.
It’s a controlled, scripted installation lifecycle.

That is why, when you install certain software via RPM, you can just launch it and the service just works.

```bash
systemctl start redis
```

## Heart of the RPM - .spec file

How does your RPM know how to build the source code and where to put the files? The magic is in the .spec file!

An RPM doesn’t guess how to build anything.
It follows instructions written in a .spec file.

The spec file tells rpmbuild:
* Where to get the source
* How to unpack it (%prep)
* How to compile it (%build)
* Where to copy files (%install)
* What files belong in the package (%files)
* What to run before/after install (%pre, %post, etc.)

It’s basically a build + packaging recipe.

Here is an example of a redis.spec file: 

```bash
Name:           redis
Version:        7.2.4
Release:        1%{?dist}
Summary:        In-memory key-value data store

License:        BSD
URL:            https://redis.io
Source0:        https://download.redis.io/releases/redis-%{version}.tar.gz

BuildRequires:  gcc
BuildRequires:  make
Requires:       systemd

%description
Redis is an in-memory key-value database supporting persistence and replication.

%prep
%setup -q

%build
make BUILD_TLS=no

%install
rm -rf %{buildroot}

# Create directories
install -d %{buildroot}/usr/bin
install -d %{buildroot}/etc/redis
install -d %{buildroot}/var/lib/redis
install -d %{buildroot}/var/log/redis
install -d %{buildroot}/usr/lib/systemd/system

# Install binaries
install -m 0755 src/redis-server %{buildroot}/usr/bin/
install -m 0755 src/redis-cli %{buildroot}/usr/bin/

# Install default config
install -m 0644 redis.conf %{buildroot}/etc/redis/redis.conf

# Install systemd unit
cat > %{buildroot}/usr/lib/systemd/system/redis.service <<EOF
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
ExecStart=/usr/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/bin/redis-cli shutdown
User=redis
Group=redis
Restart=always

[Install]
WantedBy=multi-user.target
EOF

%pre
getent group redis >/dev/null || groupadd -r redis
getent passwd redis >/dev/null || \
    useradd -r -g redis -d /var/lib/redis -s /sbin/nologin \
    -c "Redis user" redis

%post
systemctl daemon-reload
systemctl enable redis.service >/dev/null 2>&1 || :

%preun
if [ $1 -eq 0 ]; then
    systemctl disable redis.service >/dev/null 2>&1 || :
    systemctl stop redis.service >/dev/null 2>&1 || :
fi

%files
%license COPYING
/usr/bin/redis-server
/usr/bin/redis-cli
/etc/redis/redis.conf
/usr/lib/systemd/system/redis.service
/var/lib/redis
/var/log/redis

%changelog
* Tue Feb 18 2026 Anand Siva <you@example.com> - 7.2.4-1
- Initial package
```

Spec file building is its own little art form. Entire careers revolve around getting these right. So rather than pretend we’re packaging for Fedora, here’s a stripped-down example that shows what’s really happening.

### %prep — Unpack and Prepare

This is the preparation stage.

RPM extracts the source tarball into a build directory and gets it ready to compile. In most cases, %setup -q handles the extraction automatically.

Nothing is installed yet. Nothing is compiled.

This is just staging the source code.

### %build — Compile the Software

This is where the actual build happens.

For Redis, that might be as simple as:

```bash
make
```
If it were a typical autotools project, you might see:

```bash
./configure
make
```

RPM doesn’t know how to build Redis.
You tell it exactly how.

This section runs inside a controlled build environment. Whatever commands you place here must produce the binary.

### %install — Stage the Files

This is where things start to feel “magical.”

But nothing is magic.

You are not installing files onto the system.
You are copying them into a temporary directory called:

```bash
%{buildroot}
```

Think of %{buildroot} as a fake root filesystem.

If you write:

```bash
install -m 0755 src/redis-server %{buildroot}/usr/bin/
```

this just means when this RPM is installed later, I want redis-server to live in /usr/bin.
You are constructing the future filesystem layout manually.
Service files, config files, logrotate configs — they’re just files you place inside buildroot.

### %files — Declare Ownership

RPM records them in its database.

That’s how rpm -ql redis works.
That’s how clean uninstalls work.
That’s how upgrades replace the right files.

If it’s not listed in %files, it doesn’t exist to RPM.

### %pre, %post, %preun

automation that lets you

* Create system users
* Reload systemd
* Enable services
* Clean up on uninstall

---

## Not so magic is it

Just look at this section above, after reading through it, you will notice one important thing. There is no crazy logic or anything that creates that rpm.
If you look at it another way you could basically create a shell script/ansible playbook to do the same exact thing. This is exactly what many teams do.

If you had the correct files you could basically write a script that `make` install, copy the correct files in place, and run some initialization scripts.

These are one of those things that it really helps understanding the fundamentals of. Many ops people including myself used to think wow there must be some crazy technology behind creating an rpm file. Had to be some complicated program that ran in some low level language that made the software work. 

The difficult part is understanding the directives of the .spec file.

---

## Make your own magic

{{< image src="magic.gif" alt="Magic confetti" style="width:100%; max-width:100%;" >}}

Now that we understand the basics of rpms and how they are put together. What is the tool that creates these rpms? 

Source github repo for rpm - https://github.com/rpm-software-management/rpm

We are going to create our own rpm. No need to use cloud resources to use a fedora server. We will be using docker.

Here is the repo if you want to follow along.

https://github.com/anand-siva/docker-rpmbuild

Clone this repo if you want to try it out.

The first things you need to do is build the image

* selfless plug! you progress=plain so it you can read it easier: 
https://anandsiva.com/posts/no_docs_given/debugging_dockerfiles/

```bash
docker build --progress=plain -t docker-rpmbuild .

 docker images | grep rpm
docker-rpmbuild      latest       03f1c5a48069   15 seconds ago   985MB
```
In the repo I have examples files to build redis rpm

lets follow those directions:

```bash
mkdir -p rpmbuild/SOURCES
wget https://download.redis.io/releases/redis-7.2.4.tar.gz
mv redis-7.2.4.tar.gz rpmbuild/SOURCES/
cp examples/redis/SOURCES/redis.service rpmbuild/SOURCES/
cp examples/redis/SOURCES/redis.logrotate rpmbuild/SOURCES/

mkdir -p rpmbuild/SPECS
cp examples/redis/SPECS/redis.spec rpmbuild/SPECS/

docker run --rm -it \
  -v "$PWD/rpmbuild:/home/builder/rpmbuild" \
  -w /home/builder/rpmbuild \
  docker-rpmbuild \
  rpmbuild -ba SPECS/redis.spec
```
This might take a bit and there is going to be a lot of text on your screen. That is alright since it is getting the source binary and compiling it for the source OS in your dockerfile.

This is what is end result of building the rpm

```bash
Wrote: /home/builder/rpmbuild/SRPMS/redis-7.2.4-1.fc40.src.rpm
Wrote: /home/builder/rpmbuild/RPMS/aarch64/redis-7.2.4-1.fc40.aarch64.rpm
Wrote: /home/builder/rpmbuild/RPMS/aarch64/redis-debugsource-7.2.4-1.fc40.aarch64.rpm
Wrote: /home/builder/rpmbuild/RPMS/aarch64/redis-debuginfo-7.2.4-1.fc40.aarch64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.TUSuU7
+ umask 022
+ cd /home/builder/rpmbuild/BUILD
+ cd redis-7.2.4
+ /usr/bin/rm -rf /home/builder/rpmbuild/BUILDROOT/redis-7.2.4-1.fc40.aarch64
+ RPM_EC=0
++ jobs -p
+ exit 0
Executing(rmbuild): /bin/sh -e /var/tmp/rpm-tmp.gtTsFl
+ umask 022
+ cd /home/builder/rpmbuild/BUILD
+ rm -rf /home/builder/rpmbuild/BUILD/redis-7.2.4-SPECPARTS
+ rm -rf redis-7.2.4 redis-7.2.4.gemspec
+ RPM_EC=0
++ jobs -p
+ exit 0

RPM build warnings:
    bogus date in %changelog: Tue Feb 18 2026 Anand Siva <you@example.com> - 7.2.4-1

```
The final RPM is written out to 

```bash
└── rpmbuild
    ├── BUILD
    ├── BUILDROOT
    ├── RPMS
    │   └── aarch64
    │       ├── redis-7.2.4-1.fc40.aarch64.rpm
    │       ├── redis-debuginfo-7.2.4-1.fc40.aarch64.rpm
    │       └── redis-debugsource-7.2.4-1.fc40.aarch64.rpm
    ├── SOURCES
    │   ├── redis-7.2.4.tar.gz
    │   ├── redis.logrotate
    │   └── redis.service
    ├── SPECS
    │   └── redis.spec
    └── SRPMS
        └── redis-7.2.4-1.fc40.src.rpm
```

Unfortunately I am writing this on my mac, so lets launch a docker fedora image to see if it works.

Let's launch a new fedora service and see if it works

```sh
docker run --rm -it \
  -v "$PWD/rpmbuild/RPMS:/rpms:ro" \
  fedora:40 \
  bash -lc "dnf -y install /rpms/aarch64/redis-*.rpm && redis-server --version"
```

Check it out !!! It worked !!!

Complete!
Redis server v=7.2.4 sha=00000000:0 malloc=jemalloc-5.3.0 bits=64 build=2bf68951364bb24f

Let's just launch bin/bash also and check it out

```bash
docker run --rm -it \
  -v "$PWD/rpmbuild/RPMS:/rpms:ro" \
  fedora:40 \
  /bin/bash

[root@d9700d26ebb0 /]# ls /rpms/aarch64/
redis-7.2.4-1.fc40.aarch64.rpm  redis-debuginfo-7.2.4-1.fc40.aarch64.rpm  redis-debugsource-7.2.4-1.fc40.aarch64.rpm

[root@d9700d26ebb0 /]# rpm -ivh /rpms/aarch64/redis-*.rpm

[root@d9700d26ebb0 /]# redis-server /etc/redis/redis.conf
144:C 19 Feb 2026 01:13:21.097 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
144:C 19 Feb 2026 01:13:21.097 * Redis version=7.2.4, bits=64, commit=00000000, modified=0, pid=144, just started
144:C 19 Feb 2026 01:13:21.097 * Configuration loaded
144:M 19 Feb 2026 01:13:21.097 * monotonic clock: POSIX clock_gettime
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 7.2.4 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 144
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

144:M 19 Feb 2026 01:13:21.098 * Server initialized
144:M 19 Feb 2026 01:13:21.098 * Ready to accept connections tcp
```
Happy building!!!!
