# ACIT 2420 - Assignment 3: Working with Static HTMLs, Nginx, and UFW

## Introduction
This README will instruct you on setting up a Bash script that generates a static index.html file containing system information. The script will be configured to run automatically every day at 5:00 AM UTC (Coordinated Universal Time). The HTML document created by this script will also be served with an nginx web server on your Arch Linux droplet along with a firewall setup using ufw to secure your server.

## Table of Contents

## Setting Up New System User and Files
Before we begin, we need to create a system user called `webgen` with a home directory at `/var/lib/webgen` and a login shell for a non-user.

The benefit of creating a system user rather than a regular user or root is so we can separate our files and other directories from our current user to prevent malicious attacks using something like the `chown` command. As for using system user instead of root, it prevents an attack from taking advantage of the elevated privileges within our system.

### Step 1: Set Up System User

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
 
    `sudo mv ~/acit2420-assignment3p1/generate_index /var/lib/webgen/bin/generate_index`

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



## References
man useradd

https://wiki.archlinux.org/title/Users_and_groups