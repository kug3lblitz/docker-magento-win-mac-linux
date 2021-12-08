# Prerequisites

Linux, or Windows PC with WSL installed, and version 2 enabled.
Docker
Git
Local php and composer are required, within WSL
Optionally, chocolatey, a command line utility for windows that simplifies third party package installation

## Useful Commands


Start (initially build) containers
	From working directory: docker compose up -d
	
	
Stop containers (do this if you have problems, need to restart services, or before turning off or hibernating/suspending your computer)
	Available globally: 
	
	docker stop $(ps -aq)
	
	
Show running docker containers (id’s and metadata)
	Available globally: 
	
	docker ps
	
	
Sign into docker environment (php container) as root (should only need this once)
	Available globally: 
	
	docker exec -it -u root <container id> bash
	
	
Sign into docker environment (php container) as normal user
	Available globally: 
	
	docker exec -it -u root <container id> bash
	
	
Activate wsl/bash (allows use of linux utilities in windows)
	Available globally: 
	
	bash or wsl
	
	
Destroy all containers, delete all cached docker images (docker-compose and all files mounted in /src will be preserved, but internal configs will be lost - nuclear option)
	Available globally, containers must already be stopped: 
	
	docker system prune -a

# Setup

You can use powershell, but I recommend using windows terminal, from the windows store, mainly because of the built-in tab support. Also grab debian, which I’d personally recommend, or ubuntu while you’re there. 
Once installed, run wsl --set-default-version 2 . This will update/upgrade the embedded linux kernel to an actual, full linux kernel (minus an init system, kernel only), and *should* apply necessary compatibility patches for interfacing with docker. 
**now** install docker. You will probably need to reboot your computer a few times during this process. It’s also a great idea to run the little demo command that docker suggests after it’s set up - if you see something other than “could not connect” at localhost/, you’ve done the docker part of this right! To confirm that docker and WSL can communicate correctly, type the following into powershell: 
	
	wsl -d docker-desktop 

If you find yourself in a bash shell, things are working as intended. Typing exit will return you to powershell.
Refer to this repository > https://github.com/markshust/docker-magento

You will also need to create a developer account at marketplace.magento.com 
Once you’re set up, be sure to confirm your email. Next, go to my profile>marketplace>Access Keys, and generate a key pair. You will need to provide a string of your choosing to serve as a seed for the key generation. The resulting public key is your composer username, and the private key, your password.

Create a working directory, ex local_magento, and cd into it.
Run wsl in powershell, and then, in the bash shell:

	curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/onelinesetup | bash -s -- magento.test 2.4.3-p1

Hopefully, this will (and should) finish with no issues. When it’s done, run:

	bin/magento --version

If this returns a magento version, or if running bin/magento resolves correctly, then we’re in good shape. Then run:

	bin/magento sampledata:deploy
	bin/magento setup:upgrade

Next, we’ll be using these guides as reference as we add a project hostname to windows, take care of SSL (which we may not actually need), and finally, configure nginx so that we can actually use our new dev environment:

https://github.com/markshust/docker-magento/discussions/372
https://jitheshkt.medium.com/enable-ssl-on-wsl2-apache-windows-10-bcdfef71024a

Now, as an administrator, open C:\Windows\System32\drivers\etc\hosts, and if there is a line beginning with 127.0.0.1 that is not prepended by an octothorpe “#”, add one to comment it out.
Next, type 

	127.0.0.1		magento.test

On a new line. To clarify, there should be a tab character between .1 and magento.test

We are nearly done. Now list your docker containers with docker ps, and note the container id of the php container. Just like a git commit, you won’t have to type the whole thing out, just the first few unique identifiers. We will now, as root, log into the docker container, with 

	docker exec -it -u root <unique ids> bash

We’re only doing a couple of important things here, installing sudo, adding default user “app” to sudoers file, and setting a password for that user.

	apt install sudo
	passwd app

	Set the password for the default user now, it doesn’t have to be secure, since this is just for local dev, you can do “password”, whatever you want and can remember

	visudo

This will open the sudoers file.
In the file, copy/paste the line which says:

	root ALL=(ALL:ALL) ALL

and replace the second “root” with app

	app ALL=(ALL:ALL) ALL

then close and save with “:x”
 
Now you can exit and log back in without root.
  

Log into dev container as default user (app)

	sudo apt update, and if necessary, sudo apt upgrade

	sudo apt install wget libnss3-tools
go to Releases · FiloSottile/mkcert, and copy the url for the latest linux amd64 release.
in container terminal, 
	
	wget <pasted url>
	mv mkcert<tab complete> mkcert (rename file to mkcert for simplicity)
	chmod a+x mkcert
	sudo mv mkcert /usr/local/bin/
 
	mkcert magento.test (replace magento.test with whatever alias you have created in the hostfile for 127.0.0.1)
run 
	mkcert -install to start using the certificate. 

Now, for the final step. Log into the container if you aren’t already. 
If you are not now in /var/www/html, cd into it. 

	rm nginx.conf
	cp nginx.sample.conf nginx.conf

	sudo su
	cd /etc/nginx
	mkdir conf.d
	cp /var/www/html/nginx.conf conf.d/

Nginx will now need to be restarted, so you can stop the containers with docker stop $(docker ps -aq), and bring them back up with a docker-compose up -d.

Now the environment should be working with a stock magento install! Project files are accessible via the src/ subdirectory of the project working directory!
