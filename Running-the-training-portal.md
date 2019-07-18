#### Docker install
~~~~
docker pull securecodingdojo/trainingportal
~~~~

Run with the following:
~~~~
docker run -p 8081:8081 \
-e DATA_DIR=$DATA_DIR \
-e CHALLENGE_MASTER_SALT=$CHALLENGE_MASTER_SALT \
-e ENC_KEY=$ENC_KEY \
-e ENC_KEY_IV=$ENC_KEY_IV \
--volume=$DATA_DIR:/dojofiles:consistent \
securecodingdojo/trainingportal
~~~~

Training portal with local user account setup will be running at: http://localhost:8081/. You can front it with NGINX (see below) and configure the public url in config.json .

Here's a sample config.json configuration

~~~~
{
    "dojoUrl" : "YOUR_DOJO_PUBLIC_URL",
    "moduleUrls" : {
        "blackBelt":"YOUR_INSECURE_INC_PUBLIC_URL",
        "securityCodeReviewMaster":"https://trendmicro.github.io/SecureCodingDojo/codereview101/?fromPortal"
    },

    "localUsersPath" : "localUsers.json",
}
~~~~

Environment variables
~~~~
$DATA_DIR - docker volume on the host where to store the db
$CHALLENGE_MASTER_SALT - secret shared between portal and vulnerable apps to validate challenges
$ENC_KEY - seed for encryption key, (store somewhere else, like /etc/profile.d)
$ENC_KEY_IV - seed for encryption IV (store somewhere else, like /etc/profile.d)
~~~~



#### Installing on a CentOS VM

The following page covers installing the training portal on RedHat/CentOS using local flat file authentication.
> Estimated duration: 30 minutes

#### Pre-requisites
~~~~
CentOS 7 minimal installation
NodeJS v10
MySQL (MariaDB)
Nginx
~~~~
#### Installing Node

Follow instructions on this page according to your distribution: https://nodejs.org/en/download/package-manager/

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

Optional, if you'd like to prevent participants from generating their own challenge codes ;) do this:

~~~~
sudo bash -c 'echo export CHALLENGE_MASTER_SALT=YOUR_CHALLENGE_MASTER_SALT >> /etc/environment'
~~~~

NOTE:
> You have to configure the same CHALLENGE_MASTER_SALT env variable on the system where Insecure.Inc is running.

Verify changes have been applied and then reboot

~~~~
cat /etc/environment
reboot
~~~~

#### Configuring the training portal settings

Change the directory to the training portal dir and copy the config.json sample and the localUsers.json sample as the user scd.

~~~~
cd /opt/scd/SecureCodingDojo/trainingportal/
sudo -u scd cp config.json.sample config.json
sudo -u scd cp localUsers.json.sample localUsers.json
~~~~

NOTE:
> THIS IS OPTIONAL - You only need to configure encrypted settings if you plan to integrate with Slack, Google or plan to use a MYSQL DB.

Edit the `encryptConfigs.js` script with vi as the scd user.

~~~~
sudo -u scd vi encryptConfigs.js
~~~~

NOTE:
>For those that are not familiar with vi enter i to go to edit mode.

Update the `dbPass` variable. You will need to edit the `slackSecret` and `googleSecret` variables if you choose to configure any of these authentication methods.


Save the `encryptConfigs.js` file.

NOTE:
>For those that are not familiar with vi hit ESC and then enter `:wq`

Run the encryptConfigs.js file to generate the configuration.

~~~~
sudo -u scd node encryptConfigs.js 
~~~~

Copy the output of the program which should look like this:
~~~~
======= config.json ==========
config.encDbPass="NOGgYuo7lAeUhZzISsYwTw=="
config.encSlackClientSecret="NOGgYuo7lAeUhZzISsYwTw=="
config.encGoogleClientSecret="FmCdrWGdzF6ExdxD5kFPbg=="
~~~~

Delete the secrets and passwords from the encryptConfigs.js file.

Edit the config.json file with vi

~~~~
sudo -u scd vi config.json
~~~~

NOTE:
> The following settings configure the training portal using local authentication (less secure). For integrating with Slack or Google authentication check the relevant wiki pages.

Paste the corresponding values from the encryptConfigs.js output. 
The config file should look something like this.

~~~~
{
    "dojoUrl" : "YOUR_DOJO_URL",
    "moduleUrls" : {
        "blackBelt":"YOUR_INSECURE_INC_URL",
        "securityCodeReviewMaster":"https://trendmicro.github.io/SecureCodingDojo/codereview101/?fromPortal"
    },

  
    "localUsersPath" : "localUsers.json",

    "googleClientId" : "YOUR_GOOGLE_CLIENT_ID",
    "slackClientId" : "YOUR_SLACK_CLIENT_ID",
    "slackTeamId" : "YOUR_SLACK_TEAM_ID",

    "encGoogleClientSecret":"GENERATED_WITH_ENCRYPT_CONFIGS",
    "encSlackClientSecret":"GENERATED_WITH_ENCRYPT_CONFIGS"
}
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

### Next steps

At this point you have the training portal application working on CentOS. If you wish to deploy in AWS you can stop here and import your installation directory into a AWS Elastic Beanstalk environment. See the corresponding wiki pages for instructions.

You may also want to enable other authentication types such as Google or Slack. See the corresponding pages for instructions. Removing out the `localUsersPath` setting will disable the local authentication option.

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
Environment=ENC_KEY=YOUR_ENC_KEY  ENC_KEY_IV=YOUR_ENC_KEY_IV CHALLENGE_MASTER_SALT=YOUR_CHALLENGE_MASTER_SALT

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