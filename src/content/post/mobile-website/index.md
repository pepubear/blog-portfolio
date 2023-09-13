---
title: "Mobile Website Deployment"
publishDate: "17 February 2023"
description: "Deploying a simple http server on my bus ride home remotely!"
tags: ["proxmox", "linux"]
---

After setting up my Proxmox Based home network lab with Cloudflare Tunnels, I decided to, out of boredom, see if I can code a website from an ssh session on my phone during my bus ride home!

Firstly, I connected to my ssh shell using Cloudflare's Tunnel technology. This feature requires a domain, but I had a few from [Freenom](https://freenom.com), so I was good with that. Through my phone I connected to Cloudflare Tunnels under the domain " sshkali.domain.com " (not mine for privacy reasons). Under this website I had access to a mobile supported shell which ran great. NOTE: to access this website, I required Cloudflare to use email 2FA, meaning I didn't have to worry about any intruders.

While inside of my Ubuntu Container, I simply downloaded the nginx software package, and created a simple website in the `/var/www/html` directory with the commands:

```bash
apt install nginx
```

```bash
nano /var/www/html/index.html
```

## Inside my terminal

![terminal](/src/content/post/mobile-website/mobileshell.png)
![htmlcode](/src/content/post/mobile-website/mobilehtmlcode.png)
