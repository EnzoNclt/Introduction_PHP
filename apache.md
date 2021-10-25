# Foreword

In order to interact your back-end PHP code with your front-end, you start a build-in web server with the command 
    
    php -S localhost:8000

&nbsp;
What does it do ? It redirects the HTTP requests that you web browser (such as Google Chrome) are performing when you access an URL such as www.google.fr or in this case "localhost:8000"
&nbsp;

But this built-in web server that comes with PHP is only suitable to do basic web development. It can't be used to expose your website to the internet as it can't handle much concurrent HTTP requests.
&nbsp;

The is why in "productione environment" you have to use a dedicated HTTP web server such as "Apache" (or the alternative "Nginx").
&nbsp;
**There is nothing to submit in your repository for today.**

*(Tip: since there is nothing to submit, you are strongly advised to ask questions)*

**This subject is deliberately light, try to understand precisely all the aspects of a web server presented here. This can only be done through documentation and by experimentation.**

# Prerequisites

First of all, be sure you understand :
- IP addresses (local IP vs public IP)
- HTTP requests, if it is still unclear, try to take a look of the "curl" command in your terminal and install the software "postman" (https://www.postman.com) which gives you a nice user interface to do the same
- DNS, try the "ping" command in your terminal

# Apache

## Installation

Type the following to proceed with the installation process.

### Step 1: Update and Upgrade the apt tool to ensure we are working with the latest and greatest.

    sudo apt update && upgrade

&nbsp;
### Step 2: Install Apache and select Y when prompted.
    sudo apt install apache2 libapache2-mod-php
&nbsp;
### Step 3:  Now that we have installed Apache we have to start the service.
    systemctl start apache2
&nbsp;
### Step 4: Enabling Apache will automatically start the web server whenever the server is turned on.
    systemctl enable apache2

**Verify Apache by visiting the server’s IP or hostname; you’ll see Apache’s default page.**
#imageCenterCorners(./img/apache-default-page.png, 200px, 0, 0)

Step 5: As you want to use Apache with your PHP code you also need to install and enable Apache PHP module.
    sudo apt install libapache2-mod-php

## Configuration
&nbsp;

All Apache configuration files are store in Apache **/etc/apache2/** :

* apache2.conf: General configuration (formerly known as httpd.conf)
* mods-available/: Available modules
* mods-enabled/: Enabled modules
* sites-available/: Available sites (there is a default config that can be used a template)
* sites-enabled/: Enabled sites

## Mods
In order to enable the modules, use `a2enmod` (Apache 2 Enable Module, see http://manpages.ubuntu.com/manpages/trusty/man8/a2enmod.8.html) followed by the name of the module.
&nbsp;
As an exemple, there is a mod called "rewrite" that enable you to "rewrite URLs". This is what's happening when you use "short link" service like https://bitly.com
&nbsp;
To enable this mod, use the command "a2enmod" (Apache 2 Enable Mod)
    sudo a2enmod rewrite
To disable a module, use `a2dismod`.

## Sites
You need to create a configuration file for each website (check sites-available for template)

In order to activate a site, use the command `a2ensite` (Apache 2 Enable Site):

    sudo a2ensite \\<configName>

To disable a site, use the command `a2dissite`.

#warn(**NEVER** modify the content of 'mods-enabled' and 'sites-enabled' !)

## Test
&nbsp;

As there is default site already activated, try to open your browser and enter 'localhost' in URL. You’ll see this page:

#imageCenterCorners(./img/apache-default-page.png, 400px, 0, 0)

#newpage

# Basic HTML web site

## Definition
&nbsp;

If you take a look of the default site configuration, the web site folder "DocumentRoot" is defined as : **/var/www/html/**

As you can have multiple website on the same server, you need to create this configuration for each web site.

Go to this folder and take a look.

When you want to access your web site on your browser, type **localhost:80**. 80 is the default port for all webpage on the internet but it is hidden by your web browser as it is the same for every web site (when you go to www.google.fr in fact you access to www.google.fr:80)

The default "DocumentRoot" localisation ("/var/www/html/") is write-protected by your Linux (except for sudo user). You will need to specify your own foldier for each of your website.

&nbsp;

## Configuration

So, go to your apache configuration folder:

    cd /etc/apache2/sites-available

Then, we will copy the default Apache configuration usually named **000-default.conf** and create a new configuration.

**NEVER** modify the default configuration)

    sudo cp 000-default.conf my-web-server.conf

Now modify this new configuration (you need "sudo emacs" as it is write-protected) such as :


```sh
<VirtualHost *:80>
    ServerName my-web-server.fr
    ServerAlias www.my-web-server.fr
    ServerAdmin contact@my-web-server.fr
    DocumentRoot SPECIFY YOUR HTML FOLDER
    ErrorLog ${APACHE_LOG_DIR}/my-web-server.log
    CustomLog ${APACHE_LOG_DIR}/my-web-server.log combined
</VirtualHost>
```

To enable your new configuration :

    sudo a2ensite my-web-server
    sudo systemctl reload apache2

## Local DNS

You also have to edit **/etc/hosts** which is a local DNS lookup table and add the following lines (you must be root to edit this file):

* **127.0.0.1	my-web-server.fr**
* **127.0.0.1	www.my-web-server.fr**


You can now access your web site with both URL my-web-server.fr or www.my-web-server.fr in you web browser.

If you encounter a 403 Forbidden HTTP Error, please verify the permissions on your folders and all apache config files.

# Apache Security

With default or wrongly-configured apache config, you are prone to be be victim of cyber attacks.

We are now going to see how to set minimum security.#br

## Server information
&nbsp;

When your server encounters an error, and cannot process the request, Apache will by default, give information about the server’s type and version. In your browser, try to access a page that does not exist on your server, such as **http://localhost/doesnt_exists**.

#imageCenterCorners(./img/apache-not-found.png, 400px, 0, 0)

* Go to folder: /etc/apache2
* Edit the file: conf-available/security.conf
* Locate the line: ServerTokens, and change it's value to: Prod
* Locate the line: ServerSignature, and change it's value to: Off
* Restart Apache

Refresh the page that does not exist.

## Disable directory listing
&nbsp;
By default, when a file is not interpreted, apache displays the list of folders and files of the requested URL, allowing any visitor to access a part, if not all, of your files and folders.

#imageCenterCorners(./img/folders-apache.png, 200px, 0, 0)

To avoid this behavior, edit the file `/etc/apache2/apache2.conf` and transform `Options Indexes FollowSymLinks` into `Options -Indexes -FollowSymLinks`.

**/etc/apache2/apache2.conf**

Here are some explanations about the available options :

- 'Options'
    - 'Indexes' : Allow Apache to list the files on the current directory if no index file was found
    - 'FollowSymLinks' : Allow Apache to follow symbolic links
- 'AllowOverride' : Indicates which directives can be override in the .htaccess file
- 'Require all granted' : Indicates which ip or host can access the resource (here, everyone can access the directory)

## Mods
&nbsp;

### mod-security
&nbsp;
`mod-security` acts as a firewall and prevents brute-force attacks.

To install it:

#terminal(sudo apt install libapache2-mod-security2 -y
#prompt sudo systemctl restart apache2)

### mod-evasive
&nbsp;
`mod-evasive` avoids "DDOS" and "HTTP bruteforce" attacks.

To install it:

#terminal(sudo apt install libapache2-mod-evasive -y
#prompt sudo systemctl restart apache2)

## Limit HTTP request size
&nbsp;
In order to prevent the storage of your server from being parasitized by large user files, it is often appropriate to limit the file upload size of your server.

To do this, edit the global configuration file of your server or vhost and add the directive `LimitRequestBody`.

For example:
#terminalNoPrompt(
```sh
    <Directory "/home/YOUR_LOGIN/Rendu">
        LimitRequestBody 10485760
    </Directory>
```
)
## Disable TRACE HTTP Request
&nbsp;
Trace HTTP Request is enabled by default. It can allow a hacker to steal your cookies.

If this behavior is not desired (most of the time), disable it by modifying `/etc/apache2/apache2.conf` and by adding `TraceEnable off`.

## Install a firewall & protect port scans
&nbsp;
There is a pre-existing firewall on Ubuntu: ufw.

#hint(https://doc.ubuntu-fr.org/ufw)

Once installed it manages traffic to and from your server, allowing you to finely manage your server's ports and their functions. The objective is not to allow attackers to take advantage of open (allow) and unprotected (vulnerabilities) ports.

Enable this firewall with the default configuration and configure it to accept ssh, ftp, and web server connections.

Try to remove port accesses that you think are unnecessary.