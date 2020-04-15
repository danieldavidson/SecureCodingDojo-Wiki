# Pre-reqs
Platform: Javascript/Node
* Install VS Code 
* node/npm (developed with Node 10)
* git/git bash

# Setup 

Switch to your workspace location and clone the repo:

`git clone https://github.com/trendmicro/SecureCodingDojo.git`

Open the `SecureCodingDojo` directory in VS Code.

Open a Terminal. Change the directory to `trainingportal`:

`cd trainingportal`

Install npm dependencies:

`npm install`

Run `npm audit fix` to fix any vulnerabilities in 3rd party dependencies.

Setup your dojo configuration file `config.json`. `config.json.sample` contains a bunch of examples but if you need a quick config to get you started use the Docker config from /build/trainingportal

`cp ../build/trainingportal/config.json.sample config.json`

Run tests with `npm test`

Open `server.js`. Hit *F5* to debug.

Your debug portal is running on http://localhost:8081

Register a test user and login to check it out.





