Before you begin you must have a running environment of the training portal with `config.js` configured and authentication configured either via local, Slack or Google or a combination.

Check the page on `Deploying the training portal on CentOS` for an example on configuring a local environment.

### Setting up the RDS database

First step is to configure a MySQL RDS database for the training portal.

- Navigate to the RDS console
- Select either MySQL or MariaDB
- Under `Specify DB details` select the instance specifications accordingly
- Configure the db identifier, the master user name and password accordingly. (You can use the same user name and password that you have in your local environment)
- Under `Configure advanced settings > Network and Security` select an existing VPC and subnet group if you have not created one for this environment
- Choose to create a new security group. One that contains your current IP will be created.
- Choose to make the instance publicly accessible (outside the VPC)
- Under `Database options` configure the DB name (e.g. `securecodingdojodb`). Leave the DB port to 3306.
- Select other options as desired

After the DB gets created verify that you can access it from your current location by running the nmap command. You can obtain the endpoint (hostname) under the CONNECT section in the RDS database instance properties. 
~~~~
nmap YOUR_RDS_ENDPOINT -p 3306 -Pn
~~~~

### Configuring the training portal to use the RDS database

For testing purposes verify that your current running instance of the training portal can use the AWS RDS db by changing the DB_HOST environment variable (if the user name and password are not the same update config.js correspondingly)









