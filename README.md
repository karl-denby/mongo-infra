# mongo-infra-docker

Currently confirmed working/tested, others may work also but have not been verified.
- [x] Cloud Manager MongoDB Agent for Mac M1/M2/M3
- [ ] Cloud Manager MongoDB Agent for Mac Intel*
- [x] Cloud Manager MongoDB Agent for Windows x86_64 
- [x] Cloud Manager MongoDB Agent for Linux x86_64
- [x] Ops Manager for Mac M1/M2/M3
- [x] Ops Manager for Mac Intel/x86_64
- [ ] Ops Manager for Windows x86_64*
- [x] Ops Manager for Linux x86_64
- [x] Ops Manager MongoDB Agent for Mac M1/M2/M3
- [x] Ops Manager MongoDB Agent for Mac Intel
- [ ] Ops Manager MongoDB Agent for Windows x86_64*
- [x] Ops Manager MongoDB Agent for Linux x86_64

'*' should work but not tested

---

## Usage

![](ops-manager/docs/images/Example.png)

**Example 1:** 

Ops Manager and one MongoDB Agent (Make sure docker has access to **12G of RAM or more...** ...if you want to do backup testing)

1. On a Mac run `bash quick-start-mac-m1.sh` if you have an M1/M2/M3 or `bash quick-start-mac-intel.sh` if you have an Intel Mac
1. If you are running linux or windows, presumably on amd64/X86_64 run `bash quick-start-other-intel.sh`

2. The script will download, build and deploy Ops Manager, please click **Sign Up** and register your first user who will be the **Global Admin** then complete the Initial Setup screens (we have pre defined some values in conf-mms.properties, so you just need to click **Continue** until its done)

3. Once the project appears go to **Deployment >> Agents >> Downloads & Settings >> Select any operating system**
    1. On the wizard that appears click **+Generate Key**
    1. Take note of the values for
    ```
    mmsGroupId=123412341234123412341234
    
    mmsApiKey=123412341234123412341234123412341234123412341234123412341234123412341234
    
    mmsBaseUrl=http://ops.om.internal:8080
    ```
    1. Update the file `ops-manager/mongod-mms/automation-agent.config` with these values, it will be used by the node container in the next step

4. Press any key to unpause the script, it will download an Agent and start up a container with it running inside that is connected to the Ops Manager you deployed earlier, your environment is setup.

5. **Optional:** If you want a proxy you can run `docker compose up -d proxy`, it will be available on http://proxy.om.internal:3128 and can work with http or https. View container logs if you want to see what is using the proxy. It is allocated around 125mb of Memory.

6. **Optional:** If you want a load-balancer run `docker compose up -d lb`, it will be availble on http://lb.om.internal and you should set your Ops Manager Central URL to this and set `X-forwarded-for`. It uses about 125mb of Memory.

7. **Optional:** If you want to see emails sent by Ops Manager run `docker compose up -d smtp` you can then go to http://localhost:1080 to see a webui and all the emails set to `smtp.om.internal:1025` it only captures while its running, so it won't show you emails from before it ran, but if you need to do email resets or check invites/alerts. It uses about 125mb of Memory.

**Backup:**
- If you want an oplog store and block store for backup testing run `docker compose up -d oplog blockstore` they will be added to the same project, then you can use the OM ui to install standalones, then move to the Admin UI and configure them as backup targets. If you setup TLS in the project, I'd recommend setting TLS for metadata to AllowTLS so that you don't have to mess with the OM keystore (unless that is what you are testing)
- If you want to setup S3 you can do `docker compose up -d metadata` you'll need this for metadata obviously and also an actual S3 endpoint, to get that going do the following
  - Add a standalone to `metadata.om.intnernal` via the OM UI/API and confirm it is running
  - Start garage/s3 with `docker compose up -d s3`
  - It has a layout of a single node, no mirroring
  - It has 2 buckets, `oplog.s3.om.internal` and `blockstore.s3.om.internal`
  - ops-manager-backup (name of the key)
    - key id is `GK187d76754d8750c5fbbc8caf`
    - secret id is `2fb9bae487f80d70b8f93f80e1e3deeff978e76ae3625d4bae932f1fbf969358`
  - It has RWO permissions on the `oplog` and `blockstore` buckets
  - Go to **Admin >> Backup**
  - Enter `/head` and hit **Set**, then **Enable Daemon**
  - Click **Configure A S3 Blockstore**
  - Click **Advanced Setup** at the bottom (needed for region) then **Create New S3 Blockstore or Oplog**
  - Name = blockstore (or oplog if you selected S3 Oplog)
  - S3 Bucket Name = blockstore (or oplog)
  - Region override = docker
  - S3 Endpoint = http://s3.om.internal:3900
  - S3 Max Connections = 50
  - Path Style Access = Off
  - Server Side Encryption = On
  - S3 Autorization Method = Keys
  - AWS Access Key = GK187d76754d8750c5fbbc8caf
  - AWS Secret Key = 2fb9bae487f80d70b8f93f80e1e3deeff978e76ae3625d4bae932f1fbf969358
  - Datastore Type = Standalone
  - MongoDB Hostname = metadata.om.internal
  - MongoDB Port = 27017
  - Username = (If you enabled auth on the project enter your user otherwise blank)
  - Password = (If you enabled auth on the project enter your user otherwise blank)
  - Connection Options =
  - Encrypt Credentials = off
  - Use TLS/SSL = if you enabled TLS in the project set this, default off
  - New assignment Enabled = on
  - Disable proxy settings = off
  - Acknowledge = on
  - Hit Create and if everything was entered correctly, the metadata and s3 containers are running, it should complete. 

## Hints and tips:

- Ops Manager needs 8G RAM to run reliably, an Agent 2.5G, so for Monitoring/Automation your looking at giving docker 10.5G
- TLS certificates (testing use only) are available, please see [Enable TLS](/ops-manager/docs/tls-for-ops-manager.md) for more details
- Stopping/Starting Ops Manager and Containers
  - `docker compose pause` # will pause all the containers from running state
  - `docker compose unpause` # will get them all going again 
- Getting a Shell / SSH on the containers
  - `docker exec -it ops /bin/bash` runs bash as root on the **ops** container
  - `docker exec -it node1 /bin/bash` runs bash as root on the **node1** container
  - you can just look at the docker-compose.yml to see what each container is called, or you can see it in `docker ps`
  - `docker stats` is a great way to see the cpu/memory usage and limits of each container 
 
---

**Example 2:** 

3x MongoDB Agents for Cloud Manager (with systemd)

1. Create a Cloud Manager project
    1. Create a Cloud Manager project on https://cloud.mongodb.com and go to **Deployment >> Agents >> Downloads & Settings >> Select your operating system**
    2. Pick any OS and on the wizard that appears, click generate an API key
    3. Take note of the values for
    ``` 
    mmsGroupId=123412341234123412341234

    mmsApiKey=123412341234123412341234123412341234123412341234123412341234123412341234
    ```
    4. Update the file `cloud-manager/mongod-mms/automation-agent.config` with these values
2. `cd cloud-manager` and run **only 1** of these download scripts to obtain the agent for your architechture 
```
bash assets/aarch64_CM-agent.sh # if your are on M1/ARM/Aarch64
bash assets/x86_64_CM-agent.sh  # if your are on Intel Mac/Windows/Linux
```
3. **Optional: update the `cloud-manager/docker-compose.yml` file, to select the right build file, the default is aarch64 for M1/ARM/Aarch64, you can change it to x86_64 for Intel Mac/Windows/Linux on line 6, 30, 54**

4. `docker compose up -d n1cm n2cm n3cm` this will build three containers with all the tools and dependencies you need. It will install and configure the MongoDB Agent (for Cloud Manager) that you downloaded in step 2, and connect it to the group you setup in step 1. The container has systemd and behaves like an operating system and is visible in your Cloud Manager project under **Deployment >> Agents >> Servers**.

5. **Optional:** if you need more nodes you can run `docker compose up -d n4cm n5cm n6cm`, they will appear in the same project. Each uses about 2.5GB of Memory.

=======

- TLS certificates (testing use only) are available, please see [Enable TLS](/ops-manager/docs/tls-for-ops-manager.md)

---

## Changelog
- 2024-05-01 Initial run at a simplified s3 setup
- 2024-04-25 Set some defaults in conf-mms.properties so initial startup is faster, add smtp catcher, initial attempt at s3 support
- 2024-04-23 Added working nginx loadbalancer and squid proxy
- 2024-04-22 Single command needed to do everything, added oplog/blockstores/metadata with resonable sizes
- 2024-04-16 Confirmed working on ARM/M1/Aaarch64, updated docs, set aarch64 as default as most users of this project (80%) are using M1's to run test environments
- 2024-04-15 Make CM act more like the OM container, change container names so you can run OM/CM agents at the same time with no clash
- 2024-04-11 Initial x86_64 Ops Manager Proof of Concept aarch64 for Cloud Manager confirmed good on Windows/M-series mac
- 2024-04-10 Initial x86_64 Cloud Manager Proof of Concept, with an untested version for aarch64
