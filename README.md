# mongo-infra (in docker)

Goals:
- All quick spin up of mongod and related software (Agents, Connectors, Sync, CLIs, etc.)
- Support Mac (M-series and Intel), Linux (x86_64) and Windows (x86_64)
- Provide items needed for typical scenario troubleshooting, host, nslookup, ping, jq, openssl

## Usage

**Example 1:** 3 Cloud Manager docker containers (with systemd)
1. Clone this repository if you haven't done so already
1. Change to the directory that matchines your machine architechture (aarch64 for mseries, x86_64 for intel) `cd cloud-manager-aarch64`
1. Create a Cloud Manager project on https://cloud.mongodb.com and go to **Deployment >> Agents >> Downloads & Settings >> Select your operating system**
1. Pick anything in the linux family and on the wizard that appears generate an API key
1. Take note of the `mmsGroupId=123412341234123412341234` and `mmsApiKey=123412341234123412341234123412341234123412341234123412341234123412341234` values
1. Update the file `./mongod-mms/automation-agent.config` with these values, it will be used by the containers in the next step
1. `docker compose up -d` # This will build a container with the tools you need, and install and configure the MongoDB Agent (for Cloud Manager) to start with the container via systemd

## Hints and tips:
- Once the containers are running for a few seconds, they should appear in **Deployment >> Agents >> Servers** and will be configurable
- The containers have a hostname like `n1.cm.internal`, they also have an alternate name if you want to use preferred hostnames `n1.alt.internal`
- The containers have self-signed ssl certs available for use, the are in the `./certs` folder on your system and in the containers under `/certs`
- To stop one container, example n2, you would run `docker compose stop n2`
- To start one container, example n2, you would run `docker compose start n2` 
- to get a shell on a container, example n2, you would run `docker exec -it n2 /bin/bash`
- Memory limits are set in `docker-compose.yml` if you need to adjust them
- `docker compose down` will stop and delete the containers (including the agent and data dir)
- You need to clean up your cloud manager afterwards if anything is left 

---

**Example 2** x86_64 Ops Manager and 1 node in a container
1. Clone this repository if you haven't done so already
1. change to the ops-manager folder `cd cloud-manager-aarch64`
1. from this folder run this command `bash assets/x86_64_OM-7.0.4.sh` to download a 7.x AppDB and an Ops Manager 7.0.4
1. now build and start the Ops container with `docker compose up -d om`
1. Keep an eye on the startup, **it can take a few minutes for it to be ready** `docker exec -it ops tail -f /opt/mongodb/mms/logs/mms0.log`
1. When its ready (listening on 8080) you can log into `http://localhost:8080` and create the 1st user (global admin) set these options
```
URL To Access Ops Manager: http://ops.om.internal:8080
"From" Email Address: something@nowhere.com
"Reply To" Email Address: something@nowhere.com
Admin Email Address: something@nowhere.com
Transport: smtp
SMTP Hostname: localhost
SMTP Server Port: 25
Next, Next, Next, Continue
```
1. Use the default project or create a new one, then go to **Deployment >> Agents >> Downloads & Settings >> Select your operating system**
1. Pick anything in the linux family and on the wizard that appears generate an API key
1. Take note of the `mmsGroupId=123412341234123412341234`,`mmsApiKey=123412341234123412341234123412341234123412341234123412341234123412341234` and `mmsBaseUrl=http://ops.om.internal:8080`values
1. Update the file `./mongod-mms/automation-agent.config` with these values, it will be used by the node containers in the next step
1. run this command `bash assets/x86_64_OM-7.0.4-agent.sh` to download the MongoDB agent to your host from your running Ops Manager container
1. run this command to copy that agent into a container and let it connect to your Ops Manager `docker compose up -d n1`
1. After a couple of seconds go to **Deployment >> Agents >> Servers** and you'll see your server

## Hints and tips:
- You can add a Monitoring Agent and Backup on that same screen
- The containers have a hostname like `n1.cm.internal`, they also have an alternate name if you want to use preferred hostnames `n1.alt.internal`
- The containers have self-signed ssl certs available for use, the are in the `./certs` folder on your system and in the containers under `/certs`
- To stop one container, example n2, you would run `docker compose stop n1`
- To start one container, example n2, you would run `docker compose start n1` 
- to get a shell on a container, example n2, you would run `docker exec -it n1 /bin/bash`
- Memory limits are set in `docker-compose.yml` if you need to adjust them
- `docker compose down` will stop **and delete** the containers (including the agent and data dir)

## Changelog
2024-04-10 Initial x86_64 Cloud Manager Proof of Concept, with an untested version for aarch64
2024-04-11 Initial x86_64 Ops Manager Proof of Concept
           aarch64 for Cloud Manager confirmed good on Windows/M-series mac

