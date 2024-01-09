---
title: "Linux Repository Server"
publishDate: "16 December 2023"
description: "Serving Kali/ Ubuntu Packages to my Local Area Network"
tags: ["linux", "proxmox", "homelab"]
---

## Introduction

One of core functions of a network is the updating and patching process. This process is critical when it comes to the functionality of a network stack, the prevention of cybersecurity events, and (don't forget) the compliance portion.

The main way I manage auto updating and patching in my [Proxmox Server Environment](https://pepsec.com/posts/home-lab/) is through well-known software such as:

- Apt Mirror
- Unattended Upgrades
- Nginx
- Crontab

Since this is a Repository server, HTTP web hosting with **Nginx** will also need to be installed alongside **Apt-Mirror**. This will allow all servers and computers on the network to read and download any packages just like a normal Linux repository.

There are many different software and configurations that I use in this implementation but I will just go over the core neccesities for the sake of time!

### Installing Packages

For this configuration, you'll only need to worry about three total packages for your Server and Clients!

**Repository Server:**
```bash
sudo apt install apt-mirror nginx
```

**Clients:**
```bash
sudo apt install unattended-upgrades
```

## Apt Mirror

Apt Mirror is the core functionality of this configuration. Apt Mirror provides the ability to clone a remote repository (with relative ease) and implement a local repository that can be descriptively configured and managed to allow your network to recieve the correct packages at all times!

### Specifications

You will need a solid amount of storage capacity and hardware specifications. At least/ around **500GB** of Solid State Storage, **24G** of RAM, and **8** CPU Cores.

These repositories get huge and it takes alot of time to determine which repositories you will need for your network. It takes alot of research too and I am hoping this blog assists with that need a whole ton!

### Configuration

Apt Mirror comes with a basic configuration file contained in `/etc/apt/mirrors.list`

In this configuration, I am utilizing only **Ubuntu Lunar 23.04** and **Rolling Kali Linux**. That is all I need for my environment, and it is the bare minumum that I can actually support.

Take note of the Lunar and Kali apt keywords:

- main
- restricted
- universe
- multiverse
- contrib
- non-free
- non-free-firmware

```bash
############# config ##################
#
# set base_path    /var/spool/apt-mirror
#
# set mirror_path  $base_path/mirror
# set skel_path    $base_path/skel
# set var_path     $base_path/var
# set cleanscript $var_path/clean.sh
# set defaultarch  <running host architecture>
# set postmirror_script $var_path/postmirror.sh
# set run_postmirror 0
set nthreads     20
set _tilde 0
#
############# end config ##############

deb http://archive.ubuntu.com/ubuntu lunar main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu lunar-security main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu lunar-updates main restricted universe multiverse

deb http://http.kali.org/kali kali-rolling main contrib non-free non-free-firmware

clean http://archive.ubuntu.com/ubuntu
clean http://http.kali.org/kali
```

#### Ubuntu Apt Naming Scheme

The naming scheme may seem a little complicated, but it shouldn't be too hard to grasp!

- **Lunar** = Describes the version name of the Ubuntu Distribution of choice (Lunar Lobster = 23.04)
- **Main** = This contains the core set of packages that are officially supported by the distribution maintainers.
- **Restricted** = These are packages that are supported by the community, but their use might be restricted by legal or licensing issues.
- **Universe** = This includes community-maintained packages that are not officially supported by the distribution maintainers
- **Multiverse** = This includes packages that are restricted by copyright or legal issues and are not open source
- **Contrib** = This includes packages contributed by the community that are not part of the core distribution but are still officially supported.
- **Non-free** = This includes packages that are not open source and may have licensing restrictions. These packages are still provided by Kali Linux for users who may need them.
- **Non-free-firmware** = This component includes non-free firmware files. Firmware files are often necessary for certain hardware components to function correctly.

### Running

When running Apt-Mirror I use a **crontab** configuration which I will explain later in this blog, but to run Apt-Mirror manually just use the command `sudo apt-mirror`.

**CAUTION:** This may take a WHILE

```bash
repo@repository:~$ sudo apt-mirror
[sudo] password for repo:
Downloading 152 index files using 20 threads...
Begin time: Fri Dec 22 10:36:27 2023
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Fri Dec 22 10:36:47 2023

Processing translation indexes: [TTTT]

Downloading 555 translation files using 20 threads...
Begin time: Fri Dec 22 10:36:47 2023
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Fri Dec 22 10:38:59 2023

Processing DEP-11 indexes: [DDDD]

Downloading 56 dep11 files using 20 threads...
Begin time: Fri Dec 22 10:38:59 2023
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Fri Dec 22 10:39:00 2023

Processing cnf indexes: [CCCC]

Downloading 24 cnf files using 20 threads...
Begin time: Fri Dec 22 10:39:00 2023
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Fri Dec 22 10:39:00 2023

Processing indexes: [PPPP]

2.6 GiB will be downloaded into archive.
Downloading 56 archive files using 20 threads...
Begin time: Fri Dec 22 10:39:07 2023
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Fri Dec 22 10:40:04 2023

2.3 GiB in 45 files and 0 directories can be freed.
Run /var/spool/apt-mirror/var/clean.sh for this purpose.

Running the Post Mirror script ...
(/var/spool/apt-mirror/var/postmirror.sh)


Post Mirror script has completed. See above output for any possible errors.
```

#### Crontab

Crontab allows me to automatically run all the scripts I want with any sort of modification to allow myself to be assured my installation is running correctly all the time. This concept is heavily adopted by large and small enterprises in their server administration needs!

If you noticed, my Crontab configuration will also include the "**/var/spool/apt-mirror/var/clean.sh**" script to clean up Apt-Mirror.

```bash
0 3 * * * /usr/bin/apt-mirror 2>&1 | tee -a /var/log/apt-mirror.log | logger -t apt-mirror
0 4 * * * /var/spool/apt-mirror/var/clean.sh 2>&1 | tee -a /var/log/apt-mirror-clean.log | logger -t apt-mirror-clean
```

**First Line (Apt Mirroring):**

- `0 3 \* \* \*`: This specifies the schedule for the cron job. In this case, the job runs at 3:00 AM daily (0 minutes, 3 hours).
- /usr/bin/apt-mirror: The command to execute. In this instance, it runs the apt-mirror tool, which mirrors Debian or Ubuntu repositories locally.
- 2>&1: Redirects both standard output and standard error to the next command.
- tee -a /var/log/apt-mirror.log: Appends the output to the specified log file /var/log/apt-mirror.log.
- logger -t apt-mirror: Logs the output using the syslog with the tag "apt-mirror."

**Second Line (Apt Mirror Cleanup):**

- `0 4 \* \* \*`: Scheduled at 4:00 AM daily.
- /var/spool/apt-mirror/var/clean.sh: Executes the cleanup script for the apt-mirror, responsible for removing obsolete files and maintaining the mirror efficiently.
- 2>&1: Redirects both standard output and standard error to the next command.
- tee -a /var/log/apt-mirror-clean.log: Appends the output to the specified log file /var/log/apt-mirror-clean.log.
- logger -t apt-mirror-clean: Logs the output using the syslog with the tag "apt-mirror-clean."

## Unattended Upgrades

I have gone over the Repository server portion of the network stack, but what about the clients who need to update and upgrade packages? Well it is actually incredibly simple and all that is needed is some simple copy and pasting!

### Sources.list

This configuration specifies which servers and what packages to update and upgrade from! Remember the Ubuntu naming scheme, you'll notice it is quite similar to the Apt Mirror configuration.

**/etc/apt/sources.list**

```bash
deb http://10.0.0.4/archive.ubuntu.com/ubuntu lunar main restricted universe multiverse
deb http://10.0.0.4/archive.ubuntu.com/ubuntu lunar-security main restricted universe multiverse
deb http://10.0.0.4/archive.ubuntu.com/ubuntu lunar-updates main restricted universe multiverse
```

### 50unattended-upgrades

This configuration specifies the allowed origins for unattended upgrades. In this case, it allows upgrades from three different sources.

**/etc/apt/apt.conf.d/50unattended-upgrades**

```bash
Unattended-Upgrade::Allowed-Origins {
    "Ubuntu:lunar";
    "Ubuntu:lunar-security";
    "Ubuntu:lunar-updates";
};
Unattended-Upgrade::SyslogEnable "true";
Unattended-Upgrade::SyslogFacility "daemon";
```

- "**Ubuntu:lunar**": The main repository for the specified Ubuntu release (lunar in this case).
- "**Ubuntu:lunar-security**": The security updates repository for the specified Ubuntu release.
- "**Ubuntu:lunar-updates**": The updates repository for the specified Ubuntu release.

### 20auto-upgrades

**/etc/apt/apt.conf.d/20auto-upgrades**

```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

- **APT::Periodic::Update-Package-Lists "1";** = This line sets the frequency of updating the package lists. The value "1" means that the package lists will be updated daily.
- **APT::Periodic::Download-Upgradeable-Packages "1";** = This line configures how often the system should download upgradable packages. The value "1" means that it will download upgradeable packages daily.
- **APT::Periodic::AutocleanInterval "7";** = This line sets the interval for cleaning up the local repository of retrieved package files. The value "7" indicates that the cleanup will occur every 7 days. This helps in keeping the system storage from accumulating unnecessary package files.
- **APT::Periodic::Unattended-Upgrade "1";** = This line enables unattended upgrades. The value "1" means that unattended upgrades will be performed automatically. Unattended upgrades typically include security updates and other critical updates.

### System Startup

**Never Forget This!**

```bash
sudo systemctl enable unattended-upgrades
sudo systemctl restart unattended-upgrades
```

## Nginx

This config is also very simple, just copy and paste configurations and restart nginx. Fast and Simple!

### Configuration

Configuration File Contained in **/etc/nginx/sites-enabled/repository**

```http
server {
    listen 80;
    server_name repository.local;
    location / {
        alias /var/spool/apt-mirror/mirror/;
        autoindex on;
        allow all;
    }
}
```

**Link the Configuration:**

```bash
sudo ln -s /etc/nginx/sites-available/repository /etc/nginx/sites-enabled/
```

**Modify Ownership of Apt-Mirror Directory:**

```bash
sudo chown -R :www-data /var/spool/apt-mirror
sudo chmod -R 755 /var/spool/apt-mirror
```

**System Startup**

```bash
sudo systemctl restart nginx
sudo systemctl enable nginx

```

## Conclusion

That is pretty much the basics of starting a Linux Repositry! Share the Repository with your servers, computers and even friends!

As with the time I have right now, I may go in to this blog and make some new updates and changes before I ever think of having this blog used in the professional setting. Alot of changes and descriptions would need to be provisioned to allow for a much smoother implementation process!
