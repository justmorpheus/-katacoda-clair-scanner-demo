# Setting up Clair server or using locally

CoreOs Clair <https://github.com/coreos/clair>

Clair-scanner <https://github.com/arminc/clair-scanner>

You can run a dedicated Clair server with a database and use that in your ci/cd pipeline but if you want to run Clair as part of your ci/cd pipeline then you are in a surprise:

* Starting Clair from scratch takes about 20 to 30 minutes because the database needs to be filled up with CVEs
* Clair needs to access the container layers and therefore you need remote access from Clair to your build job


<br/>

---
## How to scan containers

* Start the Clair database and Clair locally or while running your job.

`docker run -d --name clair-db arminc/clair-db:latest`{{execute}}

`docker run -p 6060:6060 --link clair-db:postgres -d --name clair arminc/clair-local-scan:v2.0.8_fe9b059d930314b54c78f75afe265955faf4fdc1`{{execute}}

* Scan a vulnerable image

We need a vulnerable image. There are few vulnerable images, including Damn Vulnerable Web Application. We can use it to test clair-scanner. It is vulnerable by design

To specify this vulnerable image to be scanned, from the Ubuntu server, use the following command

`docker pull infoslack/dvwa`{{execute}}

* Run additional command to download the clair-scanner binary and perform the scan

`mkdir clair && cd clair`{{execute}}

`wget https://github.com/arminc/clair-scanner/releases/download/v12/clair-scanner_linux_386 -O /usr/local/bin/clair-scanner`{{execute}}

`chmod +x /usr/local/bin/clair-scanner`{{execute}}

* Check if clair-local-scan conatiner is running, else we will have error. This command will fetch the conatiner id and start the container if exited.

`CONTAINER="$(docker ps -a | grep -i "clair-local-scan" | awk ' { print $1 }')"`{{execute}}

`if sudo docker inspect --format="{{.State.Running}}" $CONTAINER; then docker start $CONTAINER; else echo "Container is already running, proceed with next command"; fi`{{execute}}



## Run the following command to generate the report for vulnerable infoslack/dvwa image
* This command will save the IP in the variable which is required for 

`IP="$(ip addr show ens3 | awk '$1 == "inet" {gsub(/\/.*$/, "", $2); print $2}')"`{{execute}}

* Run the clair-scanner, this will generate the report which will be saved in clair_report.json.

`clair-scanner --ip $IP -r clair_report.json infoslack/dvwa`{{execute}}

##Now our report is ready, we can read via 

`cat clair_report.json`{{execute}}

<br/>

---


