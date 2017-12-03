The following page covers installing the training portal on RedHat/CentOS using local flat file authentication.
> Estimated duration: 30 minutes

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
sudo bash -c 'echo export ENC_KEY=2b1e1c4b-51ca-4010-8dd1-458007e34a87 >> /etc/environment'
sudo bash -c 'echo export ENC_KEY_IV=a1b06d70-70a4-4af2-a79a-897c0212969a >> /etc/environment'
~~~~

Optional, if you'd like to also configure flat file encryption for the challenge codes you can do so like this:

~~~~
sudo bash -c 'echo export CHALLENGE_KEY=7041b3ed-220d-4b4d-aa4e-869eec708b4d >> /etc/environment'
sudo bash -c 'echo export CHALLENGE_KEY_IV=674a89f0-1ef2-4847-8159-7a90c6939ff2 >> /etc/environment'
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
