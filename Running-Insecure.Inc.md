> Estimated duration: 10 minutes

# Docker Image

Run the Docker container with
~~~~
docker run -p 8080:8080 \
-e CHALLENGE_MASTER_SALT=$CHALLENGE_MASTER_SALT \
 securecodingdojo/insecure.inc
~~~~

$CHALLENGE_MASTER_SALT - secret shared with the training portal

Browse to: `http://localhost:8080/insecureinc`

# VM
~~~~
Required OS - Linux
Tomcat 8
Install gcc for the C++ challenges
~~~~
Insecure.Inc is a java application. There's a prebuilt WAR in the [build](https://github.com/trendmicro/SecureCodingDojo/tree/master/build) directory that can be simply dropped in the `/webapps` folder of a Apache Tomcat 8 installation.

For instructions on how to install Tomcat you can check the following article: https://www.vultr.com/docs/how-to-install-apache-tomcat-8-on-centos-7

NOTE: 
>If you choose to install the pre-built war file the challenge codes are hardcoded. While these codes are salted with a random string and hashed for verification the algorithm could be reverse engineered allowing participants to cheat without passing the challenges. So may be a good idea to update the challenge codes.
>
>You can generate new codes using the encryptConfigs.js script in the Training Portal application. See the page on deploying the training portal on CentOS for more details.

