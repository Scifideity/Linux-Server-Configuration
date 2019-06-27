##  Linux Server Configuration

#### Blacksmith Item Catalog

Server IP : 3.219.35.196

Server URL: [http://3.219.35.196.xip.io](http://3.219.35.196.xip.io)

Software Utilized:

	Ubuntu 16.04
	Python
    Flask
    SQLAlchemy
    Git
    Nginx
    Gunicorn
    Supervisor

####  Prep on your LOCAL machine
	Create keys for grader
	ssh-keygen and call it grader_key
---
#### Browse to [Amazon Lightsail](https://aws.amazon.com/lightsail/?nc2=h_m1)

#### Create Instance

Sign up for the plan you want, lowest, currently $3.50/mo is sufficient for this project.

Instance Type: OS Only

OS : Ubuntu 16.04

Create Instance (May take a few min)
---
#### Configure your Server
Connect via SSH w/ default key (found in Accounts section) for your region

* Update and Upgrade the Server

		sudo apt-get update
        sudo apt-get upgrade

* Create user grader and install key
	```
    sudo adduser grader (pw = graderpw)
	su grader
	ubuntu@ip-x.x.x.x:~$ su grader
	grader@ip:/home/ubuntu$ cd /home/grader
	grader@ip-x.x.x.x:~$ mkdir .ssh
	grader@ip-x.x.x.x:~$ touch .ssh/authorized_keys
	grader@ip-x.x.x.x:~$ vi .ssh/authorized_keys
	grader@ip-x.x.x.x:~$ chmod 700 .ssh
	grader@ip-x.x.x.x:~$ chmod 644 .ssh/authorized_keys
	grader@ip-x.x.x.x:~$ ls -la
	 total 28
	 drwxr-xr-x 3 grader grader 4096 Jun 25 20:45 .
	 drwxr-xr-x 4 root   root   4096 Jun 25 20:42 ..
	 -rw-r--r-- 1 grader grader  220 Jun 25 20:42 .bash_logout
	 -rw-r--r-- 1 grader grader 3771 Jun 25 20:42 .bashrc
	 -rw-r--r-- 1 grader grader  655 Jun 25 20:42 .profile
	 drwx------ 2 grader grader 4096 Jun 25 20:45 .ssh
	 -rw------- 1 grader grader  644 Jun 25 20:45 .viminfo
	grader@ip-x.x.x.x:~$
	```
* Using vim or nano edit .ssh/authroized_keys (I prefer vim)

		vi .ssh/authorized_keys

        Paste contents of grader_key.pub from your LOCAL Machine

        :wq   (Save and Exit. For nano its CTRL-X, Y, <ENTER>)

* Grant ‘grader’ sudo access
	```
    ~$ sudo ls /etc/sudoers.d  <-- Find exiting username to copy
	90-cloud-init-users  README
	~$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader <-- Copy to grader
	~$ sudo ls /etc/sudoers.d
	90-cloud-init-users  grader  README
	~$ sudo vi /etc/sudoers.d/grader

	sudo ls /etc/sudoers.d  <— find existing user to copy (
	sudo cp /etc/sudoers.d/<username-from-previous-step> /etc/sudoers.d/grader
	sudo vi /etc/sudoers.d/grader
		change user name to grader
		save and quit using :wq!
	```
* Test grader ssh access using key
	```
    ssh -i grader_key grader@<server-ip>
	```
* Test grader sudo permissions
	```
    Last login: Tue Jun 25 20:51:48 2019 from x.x.x.x
	grader@ip-x.x.x.x:~$ ls /etc/sudoers.d
	ls: cannot open directory '/etc/sudoers.d': Permission denied
	grader@ip-x.x.x.x:~$ sudo ls /etc/sudoers.d
	90-cloud-init-users  grader  README
	grader@ip-x.x.x.x:~$
	```
* Change ssh to listen on port 2200
	```
    grader@ip-x.x.x.x:~$vi /etc/ssh/sshd_config
		Add ‘Port 2200’ below ‘Port 22’
		Confirm PasswordAuthentication is no(default)
		Confirm PermitRootLogin is no or prohibit-password(new default)
		:wq
    grader@ip-x.x.x.x:~$

* Restart sshd
 	```
	grader@ip-x.x.x.x:~$ sudo services sshd restart
	```

* Close SSH session and reconnect on port 2200
	* If successful
 		- Edit /etc/ssh/sshd_config
 		- Remove ‘Port 22’
 		- Save and restart sshd again
	* If unsuccessful
		- Go back a few steps and retrace looking for what went wrong

* Configure UFW (Uncomplicated FireWall)
 	```
	sudo ufw status <— check current status
	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	sudo ufw allow ssh
	sudo ufw allow 2200/tcp
	sudo ufw allow www
	sudo ufw allow ntp
	sudo ufw enable <— activates FW - BE CERTAIN BEFORE YOU ENABLE
    ```
* Get Application onto the server (git shown but you can FTP, SCP, or SFTP if you wish)
 	- Install git

 			sudo apt-get install git

 	- Configure git
 		* Configure Git :  [Source](https://www.liquidweb.com/kb/create-clone-repo-github-ubuntu-18-04/)
		* Set git global 'user.name' and 'user.email'
			```
			git config --global user.name "Your_Name"
			git config --global user.email "email_address@domain.com"
			```
        * Create project directory (I just made it a subdirectory of $HOME)
        	```
            cd $HOME
            mkdir blacksmithcatalog
            ```

     * Clone the project to the blacksmithcatalog directory using the github repository url

            git clone https://github.com/Scifideity/BlacksmithCatalog.git blacksmithcatalog

* Create the Virtual Environment
 	- Install Python3 and Virtual Environment
 	 	```
        sudo apt install python3-pip
 	 	sudo apt install python3-venv
        ```
    - Create the Virtual Environment
        ```
    	python3 -m venv blacksmithcatalog/venv
        ```
* Activate the Virtual Environment (venv)

    	cd blacksmithcatalog
     	source venv /bin/activate

NOTE: ENSURE YOU ARE IN THE (venv) FROM THIS POINT ON

	  Prompt will be prefaced with (venv)

* Install Flask Application dependancies

        pip install -r requirements.txt

* Install nginx (webserver to serve static files) and gunicorn (WSGI to handle python code)

        cd $HOME
        sudo apt install nginx
        pip install gunicorn

* Configure nginx

        sudo rm /etc/nginx/sites-enabled/default
        sudo vi /etc/nginx/sites-enabled/blacksmithcatalog

        Add the following:

            server {
 			       listen 80;
        			server_name 3.219.35.196;

        			location /static {
            		alias /home/ubuntu/blacksmithcatalog/static;
        			}

        			location / {
            			proxy_pass http://localhost:8000;
            			include /etc/nginx/proxy_params;
            			proxy_redirect off;
        			}
			}

        :wq (Save and Exit)

* Restart Nginx server

		sudo systemctl restart nginx

* OPTIONAL TEST: Run gunicorn to test

        gunicorn -w 3 application:app
        browse to public IP to test site
        CTRL-C to exit

* Install Supervisor (Monitors and restarts app)

        sudo apt install supervisor

* Configure Supervisor

		sudo vi /etc/supervisor/conf.d/blacksmithing.conf

        Add the following:

        	[program:blacksmithcatalog]
			directory=/home/ubuntu/blacksmithcatalog
			command=/home/ubuntu/blacksmithcatalog/venv/bin/gunicorn -w 3 application:app
			user=ubuntu
			autostart=true
			autorestart=true
			stopasgroup=true
			killasgroup=true
			stderr_logfile=/var/log/blacksmithcatalog/blacksmithcatalog.err.log
			stdout_logfile=/var/log/blacksmithcatalog/blacksmithcatalog.out.log
        :wq

* Make the log directory and files

        	sudo mkdir /var/log
        	sudo touch /var/log/blacksmithcatalog/blacksmithcatalog.err.log
            sudo touch /var/log/blacksmithcatalog/blacksmithcatalog.out.log

* Restart Supervisor

		sudo supervisorctl reload

#### Site will be available at : http://3.219.35.196.xip.io



---
Third Party Sites Referenced:
* [Amazon Lightsail](https://aws.amazon.com/lightsail/?nc2=h_m1)
* [ubuntu help](https://help.ubuntu.com/)
* [Corey Schafer's Tutorials](http://coreyms.com/)
* [Git Setup](https://www.liquidweb.com/kb/create-clone-repo-github-ubuntu-18-04/)
* [GitHub](http://www.github.com)
