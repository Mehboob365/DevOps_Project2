In this project we will implement a similar stack, but with an alternative Web Server – NGINX

Will download and install Git Bash like it is shown;
![image](https://user-images.githubusercontent.com/67065306/131700154-9f57ece2-4392-47fd-a00d-e1e2d7d442dc.png)

Launch GitBash
![image](https://user-images.githubusercontent.com/67065306/131700096-86a63d18-5fa3-4ec0-882a-352c148ea263.png)

Launch Git Bash and run following command:

ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>
![image](https://user-images.githubusercontent.com/67065306/131701650-26a2118b-2dc5-4268-9874-c0f936179b20.png)
  
STEP 1 – INSTALLING THE NGINX WEB SERVER
  In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package. 
  Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:

sudo apt update
  ![image](https://user-images.githubusercontent.com/67065306/131702876-8eb9bad0-336d-4d7d-9093-46b2666fb94e.png)

sudo apt install nginx
  ![image](https://user-images.githubusercontent.com/67065306/131703360-16d6d5cb-cf49-4d2c-bdc9-78c75369bb77.png)
  
To verify that nginx was successfully installed and is running as a service
  ![image](https://user-images.githubusercontent.com/67065306/131754160-5cb95a07-34c6-4187-90af-ccda60a7b882.png)

  Now it is time for us to test how our Nginx server can respond to requests from the Internet.
Open a web browser of your choice and try to access following url
         http://<Public-IP-Address>:80
  
  ![image](https://user-images.githubusercontent.com/67065306/131754420-170cedaf-c4bf-4094-9a3b-b7def90dd7fd.png)

**Step 2 — Installing MySQL**
  I am using apt to acquire and install this software:
   sudo apt install mysql-server
  
  ![image](https://user-images.githubusercontent.com/67065306/131827047-fd973be7-85ab-43e5-b93e-d0e830757c8f.png)

   When the installation is finished, it’s recommended that you run a security script that comes pre-installed with MySQL. 
   This script will remove some insecure default settings and lock down access to your database system. Start the interactive script by running:
  
  sudo mysql_secure_installation
  ![image](https://user-images.githubusercontent.com/67065306/131830055-d3026841-203d-4c8f-b008-014b08c2d06b.png)

Testing to see if I am able to log in to the MySQL console by typing:
  
   sudo mysql
  ![image](https://user-images.githubusercontent.com/67065306/131831075-bc5e4753-7334-4869-8e82-851883bebcfd.png)

 To exit the MySQL console, type:

   mysql> exit
  
   ![image](https://user-images.githubusercontent.com/67065306/131831366-62258be6-9559-4ab6-bb99-417c6bcd4296.png)

  **STEP 3 – INSTALLING PHP**
  
  So we have Nginx installed to serve content and MySQL installed to store and manage data.
  Now we can install PHP to process code and generate dynamic content for the web server.
  
  While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself 
  and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which 
  stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows
  PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.
  
  To install these 2 packages at once, run:

    sudo apt install php-fpm php-mysql
  
 ![image](https://user-images.githubusercontent.com/67065306/131833017-1f043654-54ca-4e9a-8872-9fec387812c2.png)
   When prompted, type Y and press ENTER to confirm installation.

  ![image](https://user-images.githubusercontent.com/67065306/131833168-c7251b41-aa72-45fd-aa09-1be89676dddc.png)

  We now have PHP components installed. Next, you will configure Nginx to use them.
  
  **STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR**
  
 When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single
 server. Here, we will use projectLEMP as an example domain name. On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a
 directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, 
 we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request does
 not match any other sites.

We first create the root web directory for your_domain as follows:

sudo mkdir /var/www/projectLEMP

Next, will assign ownership of the directory with the $USER environment variable, which will reference our current system user:

sudo chown -R $USER:$USER /var/www/projectLEMP
![image](https://user-images.githubusercontent.com/67065306/131835046-53fb8e45-8ab8-4970-acc8-f17e606848d2.png)

Next, will open a new configuration file in Nginx’s sites-available directory using our preferred command-line editor. Here, we’ll use nano:
sudo nano /etc/nginx/sites-available/projectLEMP
  
This will create a new blank file. Will paste in the following bare-bones configuration:

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
 
  ![image](https://user-images.githubusercontent.com/67065306/131835517-95fc1581-3fd5-448a-9e73-9b139a9aec84.png)

Here’s what each of these directives and location blocks do:

listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
root — Defines the document root where the files served by this website are stored.
index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files
        to allow for quickly setting up a maintenance landing page in PHP applications. We can adjust these settings to better suit our application needs.
server_name — Defines which domain names and/or IP addresses this server block should respond for. Pointing this directive to our server’s domain name or public IP address.
location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the
             appropriate resource, it will return a 404 error.
location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares
                    what socket is associated with php-fpm.
location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their
                   way into the document root ,they will not be served to visitors.
 
  
Activating our configuration by linking to the config file from Nginx’s sites-enabled directory:

  sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
  
This will tell Nginx to use the configuration next time it is reloaded. We can test our configuration for syntax errors by typing:

sudo nginx -t
  
  ![image](https://user-images.githubusercontent.com/67065306/131876106-73e90fd8-b370-42c0-b35c-5f83026b4318.png)

  
We also need to disable default Nginx host that is currently configured to listen on port 80, for this will run:

sudo unlink /etc/nginx/sites-enabled/default
  
Next will reload Nginx to apply the changes:

 sudo systemctl reload nginx
  
My new website is now active, but the web root /var/www/projectLEMP as we know is still empty. 
So, will create an index.html file in that location so that we can test our new server block works as expected:

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' 
$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

 Now will go to my browser and try to open my website URL using IP address:

http://<Public-IP-Address>:80
  
  ![image](https://user-images.githubusercontent.com/67065306/131893936-eeeab932-7300-4985-af8c-a4752c0871aa.png)

  
** STEP 5 – TESTING PHP WITH NGINX**
  
At this point, my LAMP stack is completely installed and fully operational.

We can test it to validate that Nginx can correctly handover .php files off to our PHP processor. 
We can do this by creating a test PHP file in our document root. 
Will now open a new file called info.php within my document root in our text editor:

sudo nano /var/www/projectLEMP/info.php

Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:

" <?php
  phpinfo(); "   without parentheses

We can now access this page in our web browser by visiting the domain name or public IP address we have set up in our Nginx configuration file, followed by /info.php:

http://`server_domain_or_IP`/info.php

![image](https://user-images.githubusercontent.com/67065306/131895611-100959bf-5ea0-49a7-8924-16d061a562d8.png)

After checking the relevant information about our PHP server through the page, it’s best to remove the file we created as it contains sensitive information about our
PHP environment and our Ubuntu server. We will use rm to remove the file:

sudo rm /var/www/your_domain/info.php

We can always regenerate this file if we need it later.


**STEP 6 – RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)**
Step 6 — Retrieving data from MySQL database with PHP

In this step will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.
At the time of this testing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. 
Will need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named example_database and a user named example_user, but we can always replace these names with different values.

First, we connect to the MySQL console using the root account:

sudo mysql

To create a new database, run the following command from your MySQL console:
mysql> CREATE DATABASE `example_database`;

Now we can create a new user and grant him full privileges on the database we have just created.

The following command creates a new user named example_user, using mysql_native_password as default authentication method. 
We are defining this user’s password as password, but one should replace this value with a secure password of your own choosing.

mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
  
Now we need to give this user permission over the example_database database:

mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';

![image](https://user-images.githubusercontent.com/67065306/131898735-d78b9f6f-62dd-435a-aaee-77aec63f886a.png)

This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on our server.

Now we will exit the MySQL shell with:

mysql> exit

We can test if the new user has the proper permissions by logging into the MySQL console again, this time using the custom user credentials:

mysql -u example_user -p

![image](https://user-images.githubusercontent.com/67065306/131899200-14509ca3-a0d4-4861-b891-657bb5e67fd1.png)

mysql> SHOW DATABASES;
This will give us the following output:

![image](https://user-images.githubusercontent.com/67065306/131899434-2b331989-39ac-4c1b-a231-fbe23dc53d65.png)

Next, we’ll create a test table named todo_list. From the MySQL console, we run the following statement:

CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );

![image](https://user-images.githubusercontent.com/67065306/131916128-77d6f4cd-5400-4c2e-9df5-84cbf007c13a.png)


We will insert a few rows of content in the test table. 

mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

To confirm that the data was successfully saved to your table, run:

mysql>  SELECT * FROM example_database.todo_list;

![image](https://user-images.githubusercontent.com/67065306/131916505-8b319865-7601-4d05-bcd4-4be4e4d305bc.png)

Repeat the next command a few times, using different VALUES:

![image](https://user-images.githubusercontent.com/67065306/131916749-d10ab0a4-676c-4ae2-bd1a-98e3ff4b6f51.png)

Now we can create a PHP script that will connect to MySQL and query for our content. 
Create a new PHP file in our custom web root directory using our preferred editor. We’ll use vi for that:

nano /var/www/projectLEMP/todo_list.php

The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. 
If there is a problem with the database connection, it will throw an exception.

Will copy this content into our todo_list.php script:

![image](https://user-images.githubusercontent.com/67065306/131918890-b5fb32fc-cc07-4f0f-af0c-a38720298f63.png)

Save and close the file when you are done editing.

We can now access this page in our web browser by visiting the domain name or public IP address configured for our website, followed by /todo_list.php:

http://<Public_domain_or_IP>/todo_list.php

we should see a page like this, showing the content we have inserted in our test table:

![image](https://user-images.githubusercontent.com/67065306/131918467-5a84b012-9682-4caa-8d7e-17aac8c0803e.png)

