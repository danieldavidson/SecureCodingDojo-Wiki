#### Docker install
~~~~
docker pull securecodingdojo/trainingportal
~~~~

Run with the following:
~~~~
docker run -p 8081:8081 \
-e DOJO_URL=http://localhost:8081 \
-e DOJO_TARGET_URL=$1 \
-e DATA_DIR=/dojofiles \
--volume=/$2:/dojofiles:consistent \
securecodingdojo/trainingportal
~~~~
Where:
$1 is the url of the Insecure.Inc web app
$2 is a host directory where to save the local files

Training portal with local user account setup will be running at: http://localhost:8081/

#### Installing on a CentOS VM

The following page covers installing the training portal on RedHat/CentOS using local flat file authentication.
> Estimated duration: 30 minutes

#### Pre-requisites
~~~~
CentOS 7 minimal installation
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

You can access the mysql console using the following command

~~~~
mysql -u root -p
~~~~

Create a user with the following command:

~~~~
CREATE USER 'securecodingdojo';
~~~~

Create a database using the following command:

~~~~
CREATE DATABASE securecodingdojodb;
~~~~

Grant your newly created user access to the secure coding dojo database with the following:

~~~~
GRANT ALL ON securecodingdojodb.* to 'securecodingdojo'@'localhost'
 IDENTIFIED BY '_dbpassword_';
~~~~


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


#### Configuring encryption key seeds and other environment variables

You can configure flat file encryption keys as environment variables in `/etc/environment`. If you need some handy GUIDs you can get them from this site: https://www.guidgenerator.com/online-guid-generator.aspx 

~~~~
sudo bash -c 'echo export ENC_KEY=YOUR_ENC_KEY >> /etc/environment'
sudo bash -c 'echo export ENC_KEY_IV=YOUR_ENC_KEY_IV >> /etc/environment'
~~~~

Optional, if you'd like to also configure flat file encryption for the challenge codes you can do so like this:

~~~~
sudo bash -c 'echo export CHALLENGE_KEY=YOUR_CHALLENGE_KEY >> /etc/environment'
sudo bash -c 'echo export CHALLENGE_KEY_IV=YOUR_CHALLENGE_KEY_IV >> /etc/environment'
~~~~

Configure the following environment variables for specifying the address of the training portal, InsecureInc target host and the database server address. 

NOTE:
> These are particularly useful if later, you choose to deploy the environment on AWS ELB. You can also edit the host names and db names directly in /trainingportal/config.js if you don't plan to deploy on AWS.

~~~~
sudo bash -c 'echo export DOJO_URL=http://`hostname -f`:8081 >> /etc/environment'
sudo bash -c 'echo export DOJO_TARGET_URL=http://insecureinchost:8080/insecureinc >> /etc/environment'
sudo bash -c 'echo export DOJO_DB_HOST=localhost >> /etc/environment'
~~~~

Verify changes have been applied and then reboot

~~~~
cat /etc/environment
reboot
~~~~

#### Configuring the training portal settings

Change the directory to the training portal dir and copy the config.js sample, the challengeSecrets.json and the localUsers.json sample as the user scd.

~~~~
cd /opt/scd/SecureCodingDojo/trainingportal/
sudo -u scd cp config.js.sample config.js
sudo -u scd cp challengeSecrets.json.sample challengeSecrets.json
sudo -u scd cp localUsers.json.sample localUsers.json
~~~~

Edit the `encryptConfigs.js` script with vi as the scd user.

~~~~
sudo -u scd vi encryptConfigs.js
~~~~

NOTE:
>For those that are not familiar with vi enter i to go to edit mode.

Update the `dbPass` variable. You will need to edit the `slackSecret` and `googleSecret` variables if you choose to configure any of these authentication methods.

Set the `regenerateSecrets` variable to true to create new secrets so that people won't be able to cheat.

NOTE:
>If you regenerate secrets you will need to update the InsecureInc app by updating the code.properties file on the tomcat deployment in 'webapps/insecureinc/classes/inc/insecure'

Save the `encryptConfigs.js` file.

NOTE:
>For those that are not familiar with vi hit ESC and then enter `:wq`

Run the encryptConfigs.js file to generate the configuration.

~~~~
sudo -u scd node encryptConfigs.js 
~~~~

Copy the output of the program which should look like this:
~~~~
======= config.js ==========
config.encDbPass="NOGgYuo7lAeUhZzISsYwTw=="
config.encSlackClientSecret="NOGgYuo7lAeUhZzISsYwTw=="
config.encGoogleClientSecret="FmCdrWGdzF6ExdxD5kFPbg=="
config.encExpressSessionSecret="GjqYjBq2sBeOSm3a4dGU/FPc8uIB9acp4VPrnIiiF6TUu4eYCidKy1FF6ZuI0ZywIcIVCJCwmZzEF/iTQUhFFfRdTXVIKezVs2QlVVbqHVcf0q5+kuJ/j/DIt/uHqkjL"
======= challengeSecrets.json ==========
Configure the CHALLENGE_KEY and CHALLENGE_KEY_IV environment variables to store the secrets encrypted!
"cwe306":"u4Jbronzpk_iQEVB5tTHMg"
...
~~~~

Delete the secrets and passwords from the encryptConfigs.js file.

Edit the config.js file with vi

~~~~
sudo -u scd vi config.js
~~~~

NOTE:
> The following settings configure the training portal using local authentication (less secure). For integrating with Slack or Google authentication check the relevant wiki pages.

Paste the corresponding values from the encryptConfigs.js output. Should look something like this.

~~~~
//rename to config.js and populate accordingly
var config = {};
config.dojoUrl = process.env.DOJO_URL;
config.dojoTargetUrl = process.env.DOJO_TARGET_URL;
config.isSecure = config.dojoUrl.startsWith("https");
config.dbHost = process.env.DOJO_DB_HOST;
config.dbName = 'securecodingdojodb';
config.dbUser = 'securecodingdojo';

config.encDbPass="NOGgYuo7lAeUhZzISsYwTw=="

//config.googleClientId = ''; //enable to configure Google authentication
config.encGoogleClientSecret = '';
config.localUsersPath = 'localUsers.json';//authentication using a password file containing users and hashed passwords, no lockout

config.encExpressSessionSecret="GjqYjBq2sBeOSm3a4dGU/FPc8uIB9acp4VPrnIiiF6TUu4eYCidKy1FF6ZuI0ZywIcIVCJCwmZzEF/iTQUhFFfRdTXVIKezVs2QlVVbqHVcf0q5+kuJ/j/DIt/uHqkjL"
config.googleOauthCallbackUrl = config.dojoUrl+"/public/google/callback";
config.slackOauthCallbackUrl = config.dojoUrl+"/public/slack/callback";

//config.slackClientId = '';//enable to configure Slack authentication
config.slackTeamId = '';
config.encSlackClientSecret = '';

module.exports = config;
~~~~

Reminder: hit ESC and enter `:wq` to save.

### Test the portal is running

Open port 8081 in the firewall with

~~~~
sudo firewall-cmd --zone=public --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
~~~~

Run server.js with:

~~~~
sudo -u scd node server.js 
~~~~

You should see the following output:

~~~~
Sun Dec 03 2017 10:15:53 GMT-0500 (EST) - Listening on 8081
Sun Dec 03 2017 10:15:53 GMT-0500 (EST) - Configured url:http://<hostname>:8081
Sun Dec 03 2017 10:15:53 GMT-0500 (EST) - Is secure:false
~~~~

Navigate to the configured url. You should now be able to register an account for yourself and start the challenges.

#### What could go wrong
* You see crypto.js exceptions. Check that you didn't leave any encrypted variables set to empty strings.
* You cannot register. Keep getting invalid captcha. The value of `encExpressSessionSecret` is probably incorrect. Generate a new one using `encryptConfigs.js` and make sure you copy the entire string.

### Next steps

At this point you have the training portal application working on CentOS. If you wish to deploy in AWS you can stop here and import your installation directory into a AWS Elastic Beanstalk environment. See the corresponding wiki pages for instructions.

You may also want to enable other authentication types such as Google or Slack. See the corresponding pages for instructions. Commenting out the `config.localUsersPath` setting will disable the local authentication option.

If you want to continue with configuring the training portal as a service see instructions below.

### Configuring the training portal as a service

Install `forever`.

~~~~
sudo npm install forever --global
~~~~

Create a service unit

~~~~
sudo vi /etc/systemd/system/trainingportal.service 
~~~~

Copy paste the following contents in the file. Don't forget to hit i for insert mode.
Also update the environment variables accordingly. The `/etc/environment` is not visible in service mode. 

~~~~
[Unit]
Description=Training portal service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/usr/bin/forever start /opt/scd/SecureCodingDojo/trainingportal/server.js
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=nodejs
Environment=ENC_KEY=<YOUR_ENC_KEY>  ENC_KEY_IV=<YOUR_ENC_KEY_IV> DOJO_URL=http://<YOUR_HOST>:8081 DOJO_TARGET_URL=http://insecureinchost:8080/insecureinc DOJO_DB_HOST=localhost

User=scd
Group=scd

[Install]
WantedBy=multi-user.target
~~~~

Save with ESC and `:wq`

Check the service is running as expected with:

~~~~
sudo systemctl start trainingportal
sudo systemctl status trainingportal
~~~~

#### What could go wrong
* You can't connect to the portal on port 8081. You probably are missing an environment variable in the Environment section of the service configuration file.

Enable the service with:

~~~~
sudo systemctl enable trainingportal
~~~~

Reboot and check that the dojo is running at startup.

### Install nginx

You probably want to enable a secure connection to the training portal with valid certificates. For this you can put nginx in front of nodejs per the instructions below.

The following article describes how to install and configure nginx on CentOS: https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7

### Generate SSL certificates for nginx

Follow the instructions in the following article to generate SSL certificates for nginx: https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-on-centos-7

### Configure the nginx reverse proxy

Edit the contents of the nginx configuration file with:

~~~~
sudo vi /etc/nginx/nginx.conf
~~~~

Replace the http configuration section, with the following:

~~~~
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        return 301 https://$host$request_uri;
    }

# Settings for a TLS enabled server.

    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/ssl/certs/nginx-selfsigned.crt";
        ssl_certificate_key "/etc/ssl/private/nginx-selfsigned.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                proxy_pass http://127.0.0.1:8081;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header X-Forwarded-Proto https;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
~~~~

Update the SELinux settings to allow nginx to accept network connections as a service.

~~~~
sudo setsebool httpd_can_network_connect 1 -P
~~~~

Remove the 8081 port in the firewall:
~~~~
sudo firewall-cmd --zone=public --permanent --remove-port=8081/tcp
sudo firewall-cmd --reload
~~~~

Update the environment variables in /etc/environment and the training portal service configuration to match the nginx SSL url. Don't forget to remove the 8081 port.

~~~~
sudo vi /etc/environment
sudo vi /etc/systemd/system/trainingportal.service
sudo systemctl daemon-reload
~~~~

Reboot. You are done.