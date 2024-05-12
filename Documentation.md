# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS 
** INTRODUCTION ** <br>
Lamp stack is an open source web development stack that is made up of: Linux, Apache, MySQL and PHP. 
In this documentation will expalin the Setup, Configuration and usage of LAMP stack. 

## Step 1: Pre-requisites 
<br>
1. Launch EC2 Instance of type t2.micro type and AMI Linux Ubuntu 24.04 LTS (HVM)  

![EC2 Instance](<Image/SC - 01.png>)
<br>
Create Key pair (Pem File format) that will be downloaded from the AWS Console while provisioning the server.  
 ![Key Pair Creation](<Image/SC - 02 Keypair.png>) 
 <Br> 
 We are going to use the pem Key downloaded to connect our EC2 instance via ssh.

2. Create Security group configuration was done with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the Internet. 
- Allow traffic on Port 22 (SSH) with source from anu IP address.
- Allow traffic on Port 443 (HTTPS) with source from anywhere on the Internet. 
![Security Group Creation](<Image/SC - 03 SG.png>)
<br>
3. Click Launch Instance and Your Instance will be Created. 
![Instance Launch result](<Image/SC - 04 Instance Lauched..png>)
<br>
4. To connect your instance via SSH, Go to your EC2 instance in your AWS console. Highlight the instance, and click connect.   
 ![SSH connect in the AWS console](<Image/SC - 05 SSH connect..png>)
 
 5. Go to your windows and `cd` into the folder where the `Pem file` was downloaded. 
 ![ AWS connect through windows Terminal](<Image/SC - 06 Connecting to AWS via bash..png>)

 ```bash 
cd downloads

chmod 400 "lampkeys.pem"

ssh -i "lampkeys.pem" ubuntu@ec2-23-20-223-76.compute-1.amazonaws.com
``` 
<br>

## Step 2: Install Apache 
1. To install Apache and update package manager 
<br>

![Apache installation](<Image/SC - 07 Apache installation..png>)

```bash
sudo apt update
```
2. Run apache2 package installation
![Apache2 Installation](<Image/SC - 08 Apache2 Package installation.png>)

```bash
sudo apt install apache2 
```
3. Verify that apache server is running on the ubuntu OS. 
![Verify apache](<Image/SC - 09  Verify Apache2.png>)

```bash
sudo systemctl status apache2
```
4. To confirm the server is running and can be accessed locally in the Ubuntu shell by running the command below. 
```bash
curl http://localhost:80
OR
curl http://127.0.0.1:80
```
![Check running server via Terminal](<Image/SC - 10 Server running on Ubuntu.png>)

5. Confirm via Public IP address if the Apache HTTP server can respond to request from the internet using the url in your browser. 

```
http://23.20.223.76:80
```
![browser apache running](<Image/SC - 11 Apache 2 running via IPv4 address.png>)

## Step3: Setting Up mySQL
1. Installing MySQL

Type the command below to install mysql into the running EC2 instance. 

```bash
sudo apt install mysql-server
```
![Install MySQL](<Image/SC - 12 installing mySQl.png>)

_When prompted, installation will confirmed by typing "Y" and then Press Enter._ 

2. Logging Into mysql Console 
to connect to the mySQL server as the administrative database user root infered by the use of sudo when running the command below. 
```bash
sudo mysql
```
Set a password for root user using mysql_native_password as default authentication method. 
The user's password was defined as "12345678"
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '12345678';
```
![Mysql password set](<Image/SC - 13 mysql psswd set.png>)

To exit the MySQL Console, _Type exit_
```bash
exit
```
## Step 4: Installing PHP 
1. PHP is the component of the set up that processes code to display dynamic content to the end user. 
The following were installed during the installation: 
- php package 
- php-mysql, a PHP module that allows PHP to communicate with SQL based databases. 
- libapache2-mod-php, to enable Apache to handle PHP files.

```bash
sudo apt install php libapache2-mod-php php-mysql
```

![Installing PHP](<Image/SC - 14 Installing PHP and PHP-mysql.png>)

2. To know the version of PHP Type 
```bash
php -v
```
![PHP version](<Image/SC - 15 PHP version.png>)
**The LAMP stack is fully Installed.**

## Step 5 - Creating a vitual host for the website using Apache 
To set up the PHP script, it's best to set up a proper Apache Virtual Host to hold the website files and folders. Virtual host allows to have multiple websites located on a single machine and i won't be moticed by multiple websites users. <br>
1. The default directory serving the apache default page is /var/www/html. Create your document directory next to the default. 
<br>
Create the directory for projectLamp using `mkdir` command
```bash
sudo mkdir /var/www/projectlamp
```
<br>
we are assigning directory ownership with $USER environment variable which references the current sysytem user. 

```bash 
sudo chown -R $USER:$USER /var/www/projectlamp  
```
![Virtual host Creation](<Image/SC - 16 Virtual Host.png>)

2. Create and open a new configuration file in apache's "sites-available" directory using vim.
```bash
sudo vim /etc/apache2/sites-available/projectlamp.conf
```
Paste in the bare-bones configuration below:
```bash
<VirtualHost *:80>
  ServerName projectlamp
  ServerAlias www.projectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/projectlamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
![VIM for conf file](<Image/SC - 17 VIM.png>)

3. Show the new file in sites-available 
```bash
sudo ls /etc/apache2/sites-available

Output:
000-default.conf default-ssl.conf projectlamp.conf
```

![List conf list](<Image/SC - 18 available conf file.png>)

_With the VirtualHost configuration, Apache will serve projectlamp using /var/www/projectlamp as its web root directory._

4. Enable the new virtual host 
```bash
sudo a2ensite projectlamp
```

![Enable VH](<Image/SC - 19 Run Project Lamp.png>)
5. Disable Apache's default website.  
The Apache’s default configuration will overwrite the virtual host if not disabled. This is required if a custom domain is not being used.
```bash
sudo a2dissite 000-default
```

![disable VH](<Image/SC - 19 Run Project Lamp.png>)

6. Ensure the configuration does not contain syntax error

To do that, Type the command below. 

```bash 
sudo apache2ctl configtest
```
![syntax error](<Image/SC - 20 Check syntax error.png>)

7. Reload apache to effect changes. 
```bash
sudo systemctl reload apache2
```
![apache reload](<Image/SC - 21 Apache reload.png>)
8. The new website is now active but the web root /var/www/projectlamp is still empty. 
create an index.html file in the location to test the virtual host work as expected. 
```bash
nano /var/www/projectlamp/index.html
```
![Index file code](<Image/SC - 22 Index file.png>)
Write the your code into the file: 
```bash
<html>
  <head>
    <title>Domain website</title>
  </head>
  <body>
    <h1>Hello Lamp</h1>

    <p>The landing page of <strong>your_domain</strong>.</p>
  </body>
</html>
```
Type `ctrl + x` to Save and `y` close the file, then go to your browser and access your server’s domain name or IP address:

cat the the working file directory to see the content.
  ![cat index file](<Image/SC - 23 cat index html.png>)


9. Open the website on the browser using the public IP address. 

```bash
http://23.20.223.76/
```
![result page](<Image/SC - 24 Result Page.png>)

## Step 6: Enable PHP on the wedsite 

With the default DirectoryIndex setting on Apache, index.html file will always take precedence over index.php file. This is useful for setting up maintenance page in PHP applications, by creating a temporary index.html file containing an informative message for visitors. The index.html then becomes the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root bringing back the regular application page. If the behaviour needs to be changed, /etc/apache2/mods-enabled/dir.conf file should be edited and the order in which the index.php file is listed within the DirectoryIndex directive should be changed.

1. Open the dir.conf file with vim to change the behaviour

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```
```
<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
![Index file PHP](<Image/SC - 25 PHP VIM.png>)

2. Relaod Apache 

Apache is reloaded so the changes takes effect.

```bash
sudo systemctl reload apache2
```
3. Create a PHP script to confirm that Apache is able to handle and process requests for PHP files. 

 index.php file was created inside the custom web root folder.

```bash
vim /var/www/projectlamp/index.php
```
Add the text below in the index.php file

```bash
<?php
phpinfo();
```
![PHP test script](<Image/SC - 26 PHP load.png>)

4. Result Page 

Refresh Your page `http://23.20.223.76/index.php`
![Final result](<Image/SC - 27 Final Result.png>)

This page provides information about the server from the perspective of PHP. It is useful for debugging and to ensure the settings are being applied correctly.

After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.

```bash
sudo rm /var/www/projectlamp/index.php
```

# Conclusion:

The LAMP stack provides a robust and flexible platform for developing and deploying web applications. By following the guidelines outlined in this documentation, It was possible to set up, configure, and maintain a LAMP environment effectively, enabling the creation of powerful and scalable web solutions.