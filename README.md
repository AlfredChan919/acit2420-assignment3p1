# ACIT 2420 - Assignment 3: Working with Static HTMLs, Nginx, and UFW

## Introduction
This README will instruct you on setting up a Bash script that generates a static index.html file containing system information. The script will be configured to run automatically every day at 5:00 AM UTC (Coordinated Universal Time). The HTML document created by this script will also be served with an nginx web server on your Arch Linux droplet along with a firewall setup using ufw to secure your server.

## Table of Contents

1. [Introduction](#introduction)
2. [Setting Up New System User and Files](#setting-up-new-system-user-and-files)
3. [Step 1: Set Up System User](#step-1-set-up-system-user)
4. [Step 2: Unit File Configuration](#step-2-unit-file-configuration)
5. [Step 3: Nginx Configuration](#step-3-nginx-configuration)
6. [Step 4: Setting Up UFW](#step-4-setting-up-ufw)
7. [Step 5: Verifying the Configuration](#step-5-verifying-the-configuration)
8. [References](#references)

## Setting Up New System User and Files
Before we begin, we need to create a system user called `webgen` with a home directory at `/var/lib/webgen` and a login shell for a non-user.

The benefit of creating a system user rather than a regular user or root is so we can separate our files and other directories from our current user to prevent malicious attacks using something like the `chown` command. As for using system user instead of root, it prevents an attack from taking advantage of the elevated privileges within our system.[1][2]

## Step 1: Set Up System User

1. Enter the following command to create a system user with a custom home directory path with a non-login user shell:

    `sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen`

-r: Creates a system account

-d: Specifies the home directory

-s: Specifies the user's login shell. In our case, `/usr/sbin/nologin` means a no interactive login user shell

Typically, the creation of a system user does not have a home directory, therefore by stating -d, we create a path to the home directory.[1]

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

-R: Recursive. It will iterate through all the files in the directory.[3]

## Step 2: Unit File Configuration

We will need a .service file in order to execute our script as well as a .timer file to execute it at 5:00 AM every day.[4]

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

If successful, we can check by entering: `systemctl status generate-index.service` and checking the logs.[4]

## Step 3: Nginx Configuration

1. Install Nginx

Enter the command to install Nginx:

`sudo pacman -S nginx`

2. Modify the `nginx.conf` file, enter:

`sudo nvim /etc/nginx/nginx.conf`

Change the `user` to webgen at the top of the file. It should look like:

        user webgen webgen;

The reason we put 2 webgen is because the first webgen states the user, and the second webgen states the usergroup.

Next, inside the http block, add:
        
        http{
            include /etc/nginx/sites-enabled/*;
        }

This will load in the configuration files. We will create the directories and its files in the next few steps.[5]

3. Checking your `nginx.conf` file

Make sure there are no errors with the file using the command:

`sudo nginx -t`

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

`listen 80` and `listen [::]:80` listens for incoming connections on both IPv4 and IPv6 on port 80 for the domain name `local_host.webgen`.[5][6]

`server_name` is the domain name we declared.

`root` signifies where we should look in order to find the file to host in response to the request that we received. So we will look at the `/var/lib/webgen/HTML` directory.

`index` signifies the file to serve to the user when the directory is accessed. In our case, the default file `index.html` will be served.

`location /` block chooses how to handle the request based on the URL. In our case, the / path means it handles requests sent to `http://local_host.webgen/`

`try_files $uri $uri/ =404` will check if the requested file is found and hosts it, if not, it will return a 404 error.[6]

By completing steps 4 and 5 and separating the server files from the config file, it allows us to have some modularity in the code and be able to turn off and on each server that we want by creating symlinks between sites-enabled and sites-available.

6. Enable Server Block

Now that we have created the server block, we will enable it by creating a symlink from the server block file to the `/etc/nginx/sites-enabled` directory. We will then add the sites-enabled files to the `nginx.conf` file to enable it. 

Enter the command: `sudo ln -s /etc/nginx/sites-available/webgen /etc/nginx/sites-enabled/`

7. Restart your nginx service

After all the changes, we must restart our Nginx service. Enter the command:

`sudo systemctl restart nginx`

Check that Nginx service is working as intended by entering:

`systemctl status nginx`

**Note:** You may receive the error "Could not build optimal types_hash". Refer to section 6.4 of the https://wiki.archlinux.org/title/Nginx wiki page.[5]

## Step 4: Setting Up UFW

We will now set up the Uncomplicated Firewall (UFW) to help secure our server.

1. Install UFW

Enter the command: `sudo pacman -S ufw`

2. Enable HTTP and SSH connections

Enter the commands: `sudo ufw allow SSH` and `sudo ufw allow http`

3. Enable SSH rate limiting

Enter the command: `sudo ufw limit SSH`

By limiting the SSH rate, the UFW will deny connections from an IP address that has attempted to initiate 6 or more connections in the last 30 seconds.[7]

4. Enable the UFW

**WARNING**: Do not do this if you have not completed steps 2 and 3, otherwise you will not be able to access the server anymore.

Enter the command: `sudo ufw enable`

**Note:** You can check your firewall status by entering the command:

`sudo ufw status`

You will be presented with an output such as:

    Status: active

    To                         Action      From
    --                         ------      ----
    SSH                        LIMIT       Anywhere
    80                         ALLOW       Anywhere
    SSH (v6)                   LIMIT       Anywhere (v6)
    80 (v6)                    ALLOW       Anywhere (v6)

If it says Status: active, then congratulations! Your firewall is now operational.

## Step 5: Verifying the Configuration

To Verify the website is working, we need to get the IP address from our droplet on Digital Ocean.

1. On DigitalOcean, open the droplets menu from the left and copy the IPv4 Address

![dropletimage](droplets_ip_image.png)

2. In your browser, enter the IP address in your URL bar and enter it. If successful, the output should look like this:

![successimage](success_image.png)

Congratulations!, you have successfully configured your Nginx web server to host a static HTML that updates every day at 5:00 AM UTC, and you also successfully secured your server using UFW!.

# References

[1] "useradd(8) - Linux manual page," *man useradd*. [Online]. Available: https://man7.org/linux/man-pages/man8/useradd.8.html. [Accessed: Nov. 19, 2024].

[2] ArchWiki, "Users and groups," *Arch Linux Wiki*. [Online]. Available: https://wiki.archlinux.org/title/Users_and_groups. [Accessed: Nov. 19, 2024].

[3] Linux Foundation, "chown - Change file owner and group," *Linux Man Pages*, 2024. [Online]. Available: https://man7.org/linux/man-pages/man1/chown.1.html. [Accessed: Nov. 24, 2024].

[4] DigitalOcean, "Understanding Systemd Units and Unit Files," *DigitalOcean Community Tutorials*. [Online]. Available: https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files. [Accessed: Nov. 24, 2024].

[5] ArchWiki, "Nginx," *Arch Linux Wiki*. [Online]. Available: https://wiki.archlinux.org/title/Nginx. [Accessed: Nov. 19, 2024].

[6] "Understanding Nginx Server and Location Block Selection Algorithms," *DigitalOcean Community Tutorials*. [Online]. Available: https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms. [Accessed: Nov. 24, 2024].

[7] ArchWiki, "Uncomplicated Firewall," *Arch Linux Wiki*. [Online]. Available: https://wiki.archlinux.org/title/Uncomplicated_Firewall. [Accessed: Nov. 24, 2024].
