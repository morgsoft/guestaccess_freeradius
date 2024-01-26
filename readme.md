# Morgsoft GuestAccess.io FreeRADIUS

##### For integration with guestaccess.io -- The best way to share & connect to secure Wi-Fi. Please see guestaccess.io for more information and setup guides. This readme will just cover some of the high level items.

##### This is an example FreeRADIUS server setup via docker-compose. Radius can be complicated, as there are several moving parts: certificates, access point, client. The goal of this project is to get you (yes, you!) setup with a working FreeRadius setup as quickly and easily as possible. This project will work for a basic self-signed certificate setup, and works for both EAP-PEAP and EAP-TLS authentication. This is designed to get you up and running quickly, but you will need to take extra precautions for production use. So for that, take this project to get you up and running and then add your desired setup from there. I spent a lot of time learning Radius, so I hope this project can save you a lot of time in return.  

## Get started 
##### This project can be started with docker-compose. The docker-compose.yml stack contains two images: 
* 2stacks/freeradius
* mysql 

##### Please clone this project and then edit .env file with your desired environment username/passwords, urls, and ports. There's an example .env file you can use incase you are unsure. You will also need to generate the certificates-- see the Certificates section inside this readme for how to generate the necessary certificates (included are simple 'make' commands powered by openssl, so its super easy to create the certificates, just follow the instructions below). Once the .env and certificates are generated, simply bring up the stack with: 
	docker-compose up -d 
##### And bring down the stack with: 
	docker-compose down

##### To use radius authentication, you will need an AP which supports Radius as well as a client device. The AP will point it's authentications to this docker-compose project running on a machine (e.g. 192.168.128.123:1812). Port 1812 is the default radius port, but can be changed this in the .env file if necessary (also FYI: 1813 is the default port for for accounting). The server running this docker-compose stack, the client device (e.g. iPhone), and the AP (e.g. Cisco Meraki MR44) will need to communication with each other to send EAP messages back and forth. For more information on how Radius and EAP message communication works, please see: https://en.wikipedia.org/wiki/RADIUS.


## Certificates
For radius, the certificates are super important. In general, there are 3 main certificate entities: 
* Certificate Authority (CA)
* Server 
* Client 

##### The following directory of this project has the necessary .cnf files for you to make the basic self-signed FreeRADIUS setup. 
	/configs/radius/certs/

#### This is an extreme oversimplification, but hopefully this is enough information for a casual developer to start with. 


##### Edit the ca.cnf config file and fill out setting for your organization. 
	make ca.pem
	make ca.der

##### Edit the server.cnf config file and fill out setting for your organization.
make server.pem
make server.csr

##### Edit the client.cnf config file and fill out setting for your user/client. These certificates can be used for EAP-TLS, or for 2 factor certificate auth with PEAP, TTLS, and FAST. 
make client.pem


##### To reset all the certificates back to default:
	rm -f *.pem *.der *.csr *.crt *.key *.p12 serial* index.txt* 
##### And then run the make commands (above) again to re-generate the certificates. 



## raddb
##### raddb is the main radius config. This is configured (by docker-compose.yml) to be stored in ./configs/radius/ There is not a right or wrong way to configure this, but the default example works for both EAP-PEAP and EAP-TLS authentication (once the certificates are generated, of course). I would recommend not touching this entire configuration unless you're ready to dive into advanced setups. 

##### To copy entire raddb config from docker container for backup:
	docker-compose cp freeradius:/etc/raddb /Users/myuser/Sites/freeradius/raddb_backup

##### To restore a raddb backup to the docker container:
	docker-compose cp /Users/myuser/Sites/freeradius/raddb_backup freeradius:/etc/raddb




## Database
##### This project uses mysql for a persistent database. For example, this is used for Users. The db & users are created.
	/configs/mysql/radius.sql

##### Logging into the raw database can be helpful, such as for managing users in a PEAP setup. The default settings to log into mysql are: 
	host: localhost:3307
	username: radius
	password: radpass 
##### Please edit these for production use in the .env and docker-compose.yml files. 

##### For example, Users can be obtained here: 
	USE radius;
	SELECT * FROM radcheck


## Sources
* https://github.com/2stacks/docker-freeradius
* https://github.com/FreeRADIUS/freeradius-server 
