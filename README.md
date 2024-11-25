# ACIT 2420 - Assignment 3: Working with Static HTMLs, Nginx, and UFW

## Introduction
This README will instruct you on setting up a Bash script that generates a static index.html file containing system information. The script will be configured to run automatically every day at 5:00 AM UTC (Coordinated Universal Time). The HTML document created by this script will also be served with an nginx web server on your Arch Linux droplet along with a firewall setup using ufw to secure your server.

## Table of Contents

## Setting Up New System User and Files
Before we begin, we need to create a system user called `webgen` with a home directory at `/var/lib/webgen` and a login shell for a non-user.

The benefit of creating a system user rather than a regular user or root is so we can separate our files and other directories from our current user to prevent malicious attacks using something like the `chown` command. As for using system user instead of root, it prevents an attack from taking advantage of the elevated privileges within our system.

## Step 1: Set Up System User

1. Enter the following command to create a system user with a custom home directory path with a non-login user shell:

    `sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen`

-r: Creates a system account

-d: Specifies the home directory

-s: Specifies the user's login shell. In our case, `/usr/sbin/nologin` means a no interactive login user shell

Typically, the creation of a system user does not have a home directory, therefore by stating -d, we create a path to the home directory.

2. Create the actual home directory for the webgen user since it doesn't exist yet by entering the command:

    `sudo mkdir -p /var/lib/webgen`

3. Create the subdirectories needed for this script using the command:

    `sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML`


4. Clone this repository to your home directory and move the generate_script using the following commands:
    
    `git clone https://github.com/AlfredChan919/acit2420-assignment3p1.git`
 
    `sudo cp ~/acit2420-assignment3p1/generate_index /var/lib/webgen/bin/`

5. Give generate_script the permission to execute the script by entering the following command:

    `sudo chmod +x /var/lib/webgen/bin/generate_index`

6. Create the index.html file in the `/var/lib/webgen/HTML` folder by entering the command:

    `sudo nvim /var/lib/webgen/HTML/index.html`

7. Change the ownership of the files and dictories to webgen using the command:

    `sudo chown -R webgen:webgen /var/lib/webgen`

-R: Recursive. It will iterate through all the files in the directory

## Step 2: Unit File Configuration

We will need a .service file in order to execute our script as well as a .timer file to execute it at 5:00 AM every day.

1. Create the unit files in the /etc/systemd/system directory as a sudouser or as root. Enter the command:

`sudo nvim /etc/systemd/system/generate-index.service`

and paste the following into the file and save it:

    [Unit]
    Description=Generate static index.html

    [Service]
    Type=oneshot
    User=webgen
    Group=webgen
    ExecStart=/var/lib/webgen/bin/generate_index

Next, create the timer unit file by entering the command:

`sudo nvim /etc/systemd/system/generate-index.timer`

Then paste the following inside and save it:

    [Unit]
    Description=Run daily at 05:00 that creates a static HTML page

    [Timer]
    OnCalendar=05:00
    Persistent=true
    [Install]
    WantedBy=timers.target

2. Enable the timer on startup by entering the command:

`sudo systemctl enable generate-index.timer`

3. Start the timer by entering the command:

`sudo systemctl start generate-index.timer`

We can check if the timer is active by typing the command: 

`sudo systemctl status generate-index.timer`

To test our service is working, type 

`sudo systemctl start generate-index.service`

If successful, we can check by entering: `systemctl status generate-index.service` and checking the logs.

## Step 3: Nginx Configuration

1. Install Nginx

Enter the command to install Nginx:

`sudo pacman -S nginx`

2. To modify the `nginx.conf` file, enter:

`sudo nvim /etc/nginx/nginx.conf`

3. Change the `user` to webgen at the top of the file. It should look like:

        user webgen webgen;

The reason we put 2 webgen is because the first webgen states the user, and the second webgen states the usergroup.

4. Create 2 new directories called "sites-available" and "sites-enabled" inside `/etc/nginx` by entering the commands:

`sudo mkdir -p /etc/nginx/sites-available` and `sudo mkdir -p /etc/nginx/sites-enabled`

5. Now, we will create a separate file for webgen inside "sites-available".

Enter the command: `sudo nvim /etc/nginx/sites-available/webgen`

Then create a new server block inside the file by copying and pasting the following:

        server {
            listen 80;
            listen [::]:80;

            server_name local_host.webgen;

            root /var/lib/webgen/HTML;
            index index.html;

                location / {
                try_files $uri $uri/ =404;
            }
        }

`listen 80` and `listen [::]:80` listens for incoming connections on both IPv4 and IPv6 on port 80 for the domain name `local_host.webgen`.

`server_name` is the domain name we declared.

`root` signifies where we should look in order to find the file to host in response to the request that we received. So we will look at the `/var/lib/webgen/HTML` directory.

`index` signifies the file to serve to the user when the directory is accessed.

By completing steps 4 and 5 and separating the server files from the config file, it allows us to have some modularity in the code and be able to turn off and on each server that we want by creating symlinks between sites-enabled and sites-available.

6. 


## References
man useradd
https://wiki.archlinux.org/title/Users_and_groups
https://wiki.archlinux.org/title/Nginx