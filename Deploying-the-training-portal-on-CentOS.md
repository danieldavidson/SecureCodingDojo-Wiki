The following page covers installing the training portal on RedHat/CentOS

#### Pre-requisites
~~~~
NodeJS v6
MySQL (MariaDB)
Nginx
~~~~
#### Installing Node

Follow instructions on this page according to your distribution: https://nodejs.org/en/download/package-manager/

#### Installing Maria DB

There's a good article on installing MariaDB on CentOS here: https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-centos-7

#### Setting up a database

You will have to create a database and a user for the secure coding dojo. Grant your secure coding dojo user full privileges on the newly created database.

The following article describes how to do that: https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql

#### Setting up a OS user for the training portal

~~~~~
sudo groupadd scd
sudo mkdir /opt/scd
sudo useradd -s /bin/nologin -g scd -d /opt/scd scd
sudo chown scd /opt/scd
~~~~~

#### Downloading the training portal

Clone the git repository as the newly created scd user.

~~~~
cd /opt/scd
sudo -u scd git clone https://github.com/trendmicro/SecureCodingDojo.git
~~~~

After running the commands you will have a new directory in /opt/scd called SecureCodingDojo.

#### Downloading the npm dependencies

Now is time to download all the 3rd party node packages used by the training portal. 

~~~~
cd SecureCodingDojo/trainingportal
sudo -u scd npm install
~~~~

