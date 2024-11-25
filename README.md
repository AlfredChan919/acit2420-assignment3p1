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


4. Clone this repository and move the generate_script using the following commands:
    
    `git clone https://github.com/AlfredChan919/acit2420-assignment3p1.git`
 
    `sudo mv /acit2420-assignment3p1/generate_index /var/lib/webgen/bin/generate_index`

5. Give generate_script the permission to execute the script by entering the following command:

    `sudo chmod +x /var/lib/webgen/bin/generate_index`

6. Create the index.html file in the `/var/lib/webgen/HTML` folder by entering the command:

    `sudo nvim /var/lib/webgen/HTML/index.html`

7. Change the ownership of the files and dictories to webgen using the command:

    `sudo chown -R webgen:webgen /var/lib/webgen`

-R: Recursive. It will iterate through all the files in the directory

## Step 2: Unit File Configuration

We will need a .service file in order to execute our script as well as a .timer file to execute it at 5:00 AM every day.

1. Move the unit files in the cloned repo to the /etc/systemd/system directory

Enter the commands:

`sudo mv /acit2420-assignment3p1/generate-index.service /etc/systemd/system/generate-index.service`

`sudo mv /acit2420-assignment3p1/generate-index.timer /etc/systemd/service/generate-index.timer`

## References
man useradd

https://wiki.archlinux.org/title/Users_and_groups