# Docker Setup
You will need the latest version of Docker. Tested on Mac with Docker Desktop 18.09.2.

# Basic Setup Instructions
You can use the `docker-compose.yml` file included with the project to spin up a basic installation of the project in a few easy steps.

- Git clone the repository
- Change directory to the repo root directory where the `docker-compose.yml` file is located
- Configure an environment variable DATA_DIR as a mount point for the dojo files. On *nix/mac modify `.bash_profile` as follows
~~~~
export DATA_DIR="/YOUR_DATA_DIR"
~~~~
- On Mac you must allow Docker access to this directory in Docker > Preferences > File Sharing
- Restart your terminal after editing or `source ~/.bash_profile` 
- Run with
~~~~
docker-compose up
~~~~

# Starting the services automatically
For starting the services automatically simply update the docker-compose.yml file to set a restart policy of always like this:
~~~~
...
  insecureinc:
    image: securecodingdojo/insecure.inc
    restart: no #change to always if you want the image to auto start
...
 trainingportal:
    image: securecodingdojo/trainingportal
    restart: always #change to always if you want the image to auto start
...
~~~~

# Configuring Public URLs
Your dojo will be running on `localhost` out of a default configuration file. 
There are two containers:
- Training portal - running on 8081
- Insecure.Inc - running on 8080

Next steps are to configure public access.

If you're in the AWS cloud you just need two different ALBs that will forward 443 traffic to 8081 and 8080 respectively.

You can also install nginx and forward traffic to the localhost port. You can find detailed information about nginx setup at this [link](https://github.com/trendmicro/SecureCodingDojo/wiki/Running-the-training-portal#install-nginx).

Once you got that setup you want to configure DNS to point to the load balancer(s).

When that is done tell the dojo what are the URLs by creating a `config.json` file in the docker volume, the $DATA_DIR env variable that you have configured earlier:

Example `$DATA_DIR/config.json` file
~~~~
{
    "dojoUrl" : "https://YOUR_PORTAL_HOST",
    "moduleUrls" : {
        "blackBelt":"https://YOUR_INSECURE_INC_HOST/insecureinc",
        "securityCodeReviewMaster":"https://trendmicro.github.io/SecureCodingDojo/codereview101/?fromPortal"
    },

    "disabledModules":["secondDegreeBlackBelt"],

    "playLinks" : {},

    "localUsersPath" : "localUsers.json"
}
~~~~
Explanations:
> The "securityCodeReviewMaster" is a static training module running directly from the Github repo

> The "secondDegreeBlackBelt" requires an AWS deployment with API gateway and Lambda which is not currently available as code. So you should disable it for now.

# Configuring CHALLENGE_MASTER_SALT

If you want to prevent participants from tricking the system and generate challenge verification codes based on the logic that is publicly available here, you must configure a secret CHALLENGE_MASTER_SALT environment variable.

- Edit `~/.bash_profile` or `/etc/environment` for server wide
~~~~
export CHALLENGE_MASTER_SALT="put something random here"
~~~~
- Restart your terminal or server and restart docker-compose.

# Configuring a MYSQL database

For large scale environments the local SQLite database included in the image will not be sufficient. You will need a full MYSQL server. The dojo will also work with MariaDB.

Once you have the server and the database created you can configure the credentials to that server in `config.json`

~~~~
{

...

    "dbHost" : "YOUR_DB_HOST",
    "dbName" : "securecodingdojodb",
    "dbUser" : "securecodingdojo",
    "encDbPass":"YOUR_ENCRYPTED_DB_PASS",

...

}
~~~~

## Generating the Encrypted DB Password.

You can generate `YOUR_ENCRYPTED_DB_PASS` with a script included in the /tools folder. But first you must create some environment variables to hold your encryption key seeds.

- Edit `~/.bash_profile` or `/etc/environment`
~~~~
export ENC_KEY="put something random here"
export ENC_KEY_IV="put something random here"
~~~~
- Restart your terminal or the server
- Edit `/tools/encryptConfigs.js`
~~~~
var dbPass="YOUR_PASSWORD";//DELETE ME WHEN DONE
~~~~
- Execute the script
~~~~
trainingportal (= node tools/encryptConfigs.js 
======= config.json ==========
"encDbPass":BAnkTgXIcshrVzSanByLOw==",

....

~~~~
- Copy the `encDbPass` to `config.json`
- Restart docker-compose

## Other Settings and Integrations.

See the wiki pages for more information on how to integrate with Google, Slack.
LDAP and ADFS options are also available. See `config.json.sample` for more settings.



