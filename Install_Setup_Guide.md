# Install and Setup Guide

> For this guide, I'm skipping the initial bootstrap of a Linux install on physical hardware and assuming you are running Linux and have an environment that comes with libvirt/virt-manager or that you can install it using your Linux distribution's package management software. You can skip the VM section if you don't want to run this is a separate VM/Linux instance. Alternatively, you can use the VM section somewhat as a guide for an initial Linux install but adapt it to a physical hardware install.

## OS and Container Environment Install

**Download RockyLinux iso:** \
https://download.rockylinux.org/pub/rocky/8/isos/x86_64/Rocky-8.6-x86_64-minimal.iso

**Create VM:** \
```sudo virt-manager``` \
(or find in menu and launch)

Click on New (circled in red) 


Choose Local install media, next, Click "Browse…" then "Browse Local" and select the iso downloaded above, and click "Choose Volume", click Forward.


If available, choose at least 4096 MiB (4GB) for the Memory, and 2 cores for the CPU, click Forward.\
Check the box to Enable storage, select the radio button to "Create a disk image" and set it to at least 50GiB. Click Forward.






Name it whatever you prefer, I’ll refer to the VM as "Health". Click the arrow next to "Network selection" and choose "Bridge", and then Finish. It may take a few minutes to create the VM and will boot into the Rock Linux install when complete.
> Note: If bridge is not available as an option, consult other documentation to setup a Bridge. NAT isn't ideal as we'll be working with a container, inside a VM, on a physical machine and we really don't want to deal with 3-levels deep of networking issues.

**Install Rocky Linux 8.6 (Very similar for Alma or RHEL 8 if you prefer one of those)** \
At the initial screen you can let it count down, or choose "Install Rocky Linux 8"






Choose your keyboard and language, click Continue.



We will make several changes to the options in this screen, starting with Network & Host Name, (setting up network first helps when working other options) then at the top left and working down, then center and down, and finally right column and down:

- Network and Host Name – Choose something appropriate here, we'll refer to this hostname as "health1". Click Apply. On the right, if not already enabled, slide the slider next to Ethernet to "On" to "plug in" the VM to the network. Click Done.

- Keyboard – Already done in initial screen, make changes here if necessary.

- Language Support – Already done in initial screen, make changes here if necessary.

- Select Time & Date and choose the closest Region/City near you that has your same timezone/Daylight Savings restrictions. Slide the slider right to enable Network Time if not already enabled. Click Done.

- Skip the Root Password setup to disallow direct login as root (good idea).

- Click on User Creation. Create a privileged user by checking the box "Make this user administrator". This is separate from the "nightscout" user we’ll use later. This is your interactive login you'll use to perform administrator-like functions, such as creating the unprivileged "nightscout" user later on. Set a password you'll remember that hopefully isn't too easy to guess. Click Done (Click Done twice if you set too weak of a password).

- Installation Source – Click the radio button for "On the network" and leave the dropdown on Closest mirror. Click done, and wait for the installer to automatically find the closest mirror.

- Software Selection - Select "Server" on the left, on the right choose: Guest Agents, Container Management, and Headless Management. Click Done.

- Installation Destination – Don’t make any changes, just click on it, and then click Done.

- KDUMP – Uncheck the box for Enable kdump. Click Done.



Click Begin Installation and take a walk around the block while it finishes. Click Reboot System in lower right.

After the reboot, login with your privileged user created during install.

**Create An Unprivileged User** \
You can name it nearly anything (don’t use spaces or special characters), nightscout may be most appropriate. I will use joeuser just as an example: \
```sudo useradd joeuser```


> Note: If you forget to use sudo before commands that require administrative privileges, you will get "Permission denied" errors.

Set a password for your “nightscout” account. 

**Prepare The System** \
Install the EPEL repository: \
```sudo dnf install epel-release -y```

Install podman-compose: \
```sudo dnf install podman-compose -y```


Disable root login over SSH \
```sudo sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config``` \
```sudo systemctl restart sshd.service```

Enable non-root users to bind to specific, standard ports. Enable ping for everyone: \
```
echo "net.ipv4.ip_unprivileged_port_start=443" | sudo tee -a /etc/sysctl.d/10-nightscout.conf
echo "net.ipv4.ip_unprivileged_port_start=80" | sudo tee -a /etc/sysctl.d/10-nightscout.conf
echo "net.ipv4.ping_group_range=0 $MAX_UID" | sudo tee -a /etc/sysctl.d/10-nightscout.conf
```

Disable SELinux and Firewalld. 
> A note on security: SELinux and Firewalld are important pieces of security software and go a long way to keeping your system more hardened/safer. If you have the time to learn how to use them appropriately, I highly recommend you do so instead of disabling them in this step. However for simplicity of the guide and to not make things overly complex on the system for novice users, this guide is simply going to disable them.

```sudo systemctl disable --now firewalld``` \
```getenforce``` \
```sudo setenforce 0``` \
```getenforce``` \
```sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config```



Restart the virtual machine: \
```sudo reboot``` \
After boot up, log into your “nightscout” account.

> Note: For the remainder of this setup I recommend using a remote shell instead of the virt-manager console window. You can copy and paste easier, and the virt-manager console can do weird things to your mouse and keyboard capture that can make using it unpredictable. I won’t cover setting up SSH or using an SSH client to connect, there are many good tutorials out there on using SSH if you choose to switch to a remote shell at this point. It is not required to switch from the virt-manager console to a remote shell, it  just adds a large amount of convenience.

**Setup Podman/Docker** \
Create/Pull Docker Compose file from main project for reference \
https://github.com/nightscout/cgm-remote-monitor/blob/master/docker-compose.yml \
Rename file to docker-compose.yml.reference to ensure it doesn’t interfere with the actual docker-compose.yml file we’ll use later.
 
Or grab file with curl: \
```curl https://raw.githubusercontent.com/nightscout/cgm-remote-monitor/master/docker-compose.yml -o docker-compose.yml.reference```


Grab the modified docker-compose.yml, from the project page where this guide lives, to use as our working/main file: \
```curl https://raw.githubusercontent.com/t1diotac/self-hosted-nightscout/main/docker-compose.yml -o docker-compose.yml``` \
And the environment file: \
```curl https://raw.githubusercontent.com/t1diotac/self-hosted-nightscout/main/env-file -o env-file``` \
Copy or link the env-file to .env (we could also just rename the env-file, but since files that start with a dot are hidden, the obvious visual will go away, so keep that in mind) \
```ln -s env-file .env```

Configure Nightscout environment variables for your needs. See docker-compose.yml, docker-compose.yml.reference, and env-file/.env for more info and examples. https://github.com/nightscout/cgm-remote-monitor#environment \
```nano docker-compose.yml``` \
[make changes] \
```CTRL+X``` \
(to save and exit)

```nano .env``` \
[make changes] \
```CTRL+X``` \ 
(to save and exit)

> Note: We'll be using Cloudflare as our tunnel from our home/private network to the Internet, and as a nice bonus, they manage SSL (HTTPS). Because of this, the traefik-related information from docker-compose.yml.reference has been removed and replaced with the cloudflared section in our working docker-compose.yml

## Cloudflare Registrar/Website and Cloudflare Tunnel

After completing the below steps to register a domain with Cloudflare, the CloudflareD container will create the tunnel between your self-hosted Nightscout site and Cloudflare:

CloudflareD Container info: \
https://hub.docker.com/r/cloudflare/cloudflared

Sign up for Cloudflare (free account) \
https://www.cloudflare.com/plans \
Register a domain (currently ~$10/yr for .com)



Create a Website within Cloudflare Dashboard with the same name as the domain you just registered. \
https://dash.cloudflare.com


Click on Access, go to Cloudflare Zero Trust \
https://one.dash.cloudflare.com/


**Tunnel** \
Click on Access → Tunnels

Click "Create a tunnel"

Name the tunnel


Click on "Docker" (next to Red Hat), Copy line in the box "Install and run connector": \
```docker run cloudflare/cloudflared: latest tunnel …```


Paste into working docker-compose.yml downloaded/created previously, see cloudflared section in docker-compose.yml for example, important piece is random characters after --token.


Save, Back to Tunnels Main. This is where you'll watch "Status" to turn green and change to "Active" when container is started and running. 

## Bring It All Together – Podman + SystemD
> Note: The default container management system on Red Hat-based systems is Podman, the remaining instructions are specific to using Podman with an unprivileged user account. Some things are similar when using Docker, but it isn’t completely interchangeable/seamless. Consult Docker documents if you are using Docker instead of Podman.

Start the containers:
Ensure you are in the directory containing the docker-compose.yml file by typing "ls" (that's LS without quotes) to list the files in the current directory. Double check any necessary changes are already made to the docker-compose.yml to fit your needs, then: \
```podman-compose up -d```

**Moment of Truth** \
After a few minutes you should see your tunnel status turn green in your Cloudflare One Tunnel dashboard. Open a new browser tab and enter your site name (https://subdomain.domain.com) and (hopefully) bask in your self-hosted glory.

**Persist After Logout And Reboot** \
Podman requires a simple tweak to the user account to keep a process running after you logout. It also requires SystemD services to gracefully bring down services on shutdown/reboot and bring them back up upon boot up. To keep services running after logout: \
```loginctl enable-linger nightscout``` \
Replace "nightscout" with whatever you named your unprivileged user that is running the nightscout container. \
To setup the containers to properly, survive a reboot, ensure you are in the directory with the docker-compose.yml, then: \
```
docker-compose stop
podman generate systemd --new --files --name cloudflared
podman generate systemd --new --files --name mongo
podman generate systemd --new --files --name nightscout
podman rm mongo nightscout cloudflared
mkdir -p ~/.config/systemd/user
mv container-* ~/.config/systemd/user
systemctl --user daemon-reload
systemctl --user enable --now container-mongo.service
systemctl --user enable --now container-nightscout.service
systemctl --user enable --now container-cloudflared.service
```

Pat yourself on the back, you're there! Assuming things are working, you're now hosting your own nightscout site that will survive you logging out, and even rebooting the server. If things aren't working, see the Troubleshooting section for common issues.

## Troubleshooting

**Tunnel never goes active/turns green.** \
Ensure you have the entire token copied and pasted without any line breaks, didn't accidentally double paste, and that the token matches what is setup in the tunnel configuration. If DNS isn't working correctly on your network, you may need to specify the internal IP of your VM instead of the hostname in the tunnel configuration on Cloudflare.

**I get Bad Gateway 502 when I visit my new site.** \
This is often the case when NightScout cannot connect to Mongo. If you’ve changed credentials or database names for Mongo, ensure all of the changes are correctly in place where necessary. The scope of changing Mongo is outside of this guide.

**I successfully get to my new site, but I have no data showing.** \
You may need to troubleshoot whichever uploader you are using. Refer to the official documentation for more info on troubleshooting your uploader. 
For Dexcom, ensure you have “bridge” in your ENABLE line, and ensure your username and password are correct and work using something else like the Clarity app.

**I don’t quite understand what I just did, please explain how each piece fits together in better detail.** \
I'll try to keep a balance between too much detail and too little. But here goes: \
At the lowest level, your computer is running software that emulates another computer – that’s the VM or virtual machine. That emulated computer is running simpler/smaller, more “contained” computer-like units called containers that have the Cloud apps embedded in them. That emulation layer is Podman, and it’s running 3 basic apps – NightScout, Mongo, and Cloudflared – that communicate with each other over an internal network specific to just those apps. To simplify creating the necessary environment for those apps to communicate, we use a Compose file whose spec was developed by Docker, but Podman is able to also use, thus the name docker-compose.yml for the file. Podman-compose (written after docker-compose to perform similar functions) also helps remove several typing steps, move some configuration to a separate file to make maintainability and portability easier (easier for me to share files generically without giving my username/passwords/configuration to people), and redeploy things faster/easier if I need to change hardware or otherwise move the setup. We don’t have to forward ports in our physical home router because CloudflareD sets up a tunnel directly from the container to Cloudflare who then essentially “mirrors” our internal site setup in the tunnel. Visually, it might look something like this: \
```
===============================VM===================================
           ==========================Podman==========================
             Mongo < ---- > NightScout < ------ > CloudflareD < ---- > Cloudflare < ---- > Internet
 ```

**I’m up, things work, but what happens when I need to update?** \
Containers make updating very streamlined, compose files streamline containers. To update: \
Login as your unprivileged user. In the directory that contains your docker-compose.yml (if you performed the steps above as laid out, that will be the directory you are in immediately after logging in) run: \
First stop and remove your containers: \
```
systemctl --user stop container-cloudflared.service
systemctl --user stop container-nightscout.service
systemctl --user stop container-mongo.service
```

Then update: \
```podman-compose pull```

Finally start your containers back up: \
```
systemctl --user start container-mongo.service
systemctl --user start container-mongo.service
systemctl --user start container-mongo.service
```
Done.
