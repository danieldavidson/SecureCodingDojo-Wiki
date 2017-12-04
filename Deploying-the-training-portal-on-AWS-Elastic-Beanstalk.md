Before you begin you must have a running environment of the training portal with `config.js` configured and authentication configured either via local, Slack or Google or a combination.

Check the page on `Deploying the training portal on CentOS` for an example on configuring a local environment.

### Setting up the RDS database

First step is to configure a MySQL RDS database for the training portal.

- Navigate to the RDS console
- Select either MySQL or MariaDB
- Under `Specify DB details` select the instance specifications accordingly
- Configure the db identifier, the master user name and password accordingly
- Under `Configure advanced settings > Network and Security` select an existing VPC and subnet group if you have not created one for this environment
- Under `Database options` configure the DB name (e.g. `securecodingdojodb`). Leave the DB port to 3306.
- Select other options as desired




