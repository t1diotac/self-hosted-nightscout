# self-hosted-nightscout

Files:
- README.md - This file
- docker-compose.yml - This is a modified docker-compose.yml originally obtained from <a href="https://github.com/nightscout/cgm-remote-monitor" target="_blank">nightscout/cgm-remote-monitor</a> to replace traefik with cloudflared and some minor other changes.
- env-file - This is the .env file, rename to .env and make chanages specific to your setup and needs.
~~ - Linux_VM-NightScout-Cloudflare-Podman-Install-Setup-Guide.odt - The guide. Ideally I should move this to the Wiki or to Markdown, but for now it's in Libre/OpenOffice ODT format. ~~
- Install_Setup_Guide.md - The **updated** fresh, new guide using Markdown.

# Summary
The guide walks through setting up a Linux Kernel Virtual Machine, KVM, using libvirt's virt-manager, and installs Rocky Linux on the VM. Rocky Linux is based on Red Hat, and Red Hat defaults to using Podman. <br />
The guide walks through setting up various settings in the OS, including Podman, and podman-compose. podman-compose is a project that uses docker-compose.yml files for Podman-based systems. <br />
After walking through getting the docker-compose.yml in place for NightScout, Mongo, and a Cloudflare tunnel, it walks through getting a domain registered on Cloudflare, setting up a free account there for getting your website mirror in place, and creating the tunnel to expose your self-hosted Nightscout to the Internet.
