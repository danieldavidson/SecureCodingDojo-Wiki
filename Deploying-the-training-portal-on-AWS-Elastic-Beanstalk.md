Before you begin, you must have a running environment of the training portal with `config.js` configured and authentication configured either via local, Slack or Google or a combination.

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

NOTE:
>You can also use the MySQLWorkbench application to verify connectivity, backup and inspect database.

#### Configuring your current instance of the training portal to use the RDS database

For testing purposes verify that your current running instance of the training portal can use the AWS RDS db by changing the DB_HOST environment variable (if the user name and password are not the same update config.js correspondingly)


### Setting up the Elastic Beanstalk environment

- Navigate to the Elastic Beanstalk console and create a new application
- Create a new environment
- Choose `web server environment`
- Choose a convenient domain. Copy this you will need it later.
- Choose a preconfigured platform: `Node.js`
- Zip the contents of the `trainingportal` directory (do not zip the folder)
- Select `Upload your source code` and point to the zip
- Select `Configure more options`
- Select the `High availability` configuration
#### Software settings
- Under `Software` click `Modify` and copy all the training portal environment variables from your environment.
Make sure the DOJO_URL matches the domain you have chosen for your environment and that the DB_HOST matches the endpoint of the RDS instance you have created.
~~~~
ENC_KEY=<YOUR_KEY>
ENC_KEY_IV=<YOUR_IV>
DOJO_URL=https://<your beanstalk FQDN>
DOJO_TARGET_URL=http://insecureinchost:8080/insecureinc
DOJO_DB_HOST=localhost
~~~~
#### Load Balancer settings
- Under `Load Balancer` enable the secure port and select an available certificate. Disable port 80.

NOTE:
>If a certificate or domain has not been reserved for you can import a self signed certificate or a certificate signed by a different authority using the AWS `Certificate Manager`

NOTE 2:
> You can use the following command to configure a self-signed cert with openssl
`openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem`

NOTE 3:
>If you choose not to setup SSL at this time make sure you configure the DOJO_URL environment variable accordingly
#### Security settings
- Under `Security` select an SSH key to login into the ec2 instance
#### Network settings
- Under `Network` select the same VPC with the RDS instance VPC. Select the same subnets for the instance and the balancer with the RDS DB subnet. 
- Select an applicable security group. If none is defined, define one unde `EC2 > Security Groups`
- Save and verify all settings then hit `Create Environment`

NOTE:
>There's no need to configure a `Database` we have configured a DB independently.










