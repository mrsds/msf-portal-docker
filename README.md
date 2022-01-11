# msf-portal-docker
README by Kevin Gill, Josh Rodriguez, Robert Tapella, and Kathryn Yu

### Setup (local instance)
#### Prerequisites
- Have access to a terminal, where the majority of the commands below will be run
- Have Git and Docker installed on your machine

#### Initialize the submodules
Run `git submodule init`, then `git submodule update` to update the submodules.

This project relies on three submodules:
- msf-be: The backend code
- msf-static-layers: Layer files served by the backend
- msf-ui: The frontend code of the Methane Portal webapp

#### Set Variables
Start by setting the proper variables. Edit the file `/msf-ui/src/config.js` to point to where the backend code is being run. The default value for this is `https://localhost:3001/server`. This should be equivalent to the base URL of the site with `/server` appended to its end. **NOTE!** Leave the `/server` part at the end of the URL, as that maps to the path that's proxied to `msf-be`. 

Edit the file `msf-be/src/msfbe/config.ini` on the line that starts with `s3.proxyurl=` to point to the correct URL of the proxy server being used to serve AWS image files. The default value for this is `https://localhost/server/image?output=PNG&item=`.

#### Save certs
In the `/certs` folder, place the `methane.jpl.nasa.gov.crt` and `methane.jpl.nasa.gov.key` files. This is a SSL protected web app so these certificates are expectd to exist. Self signed certificates may be used for testing locally.

#### Configure the environment if necessary.
The `env.conf` file has envvars that are sent to the `msf-be` instance. These include the Postgres database credentials and port (each Postgres related envvar begins with prefix `PG`), and the envvar that stores the URL of the S3 bucket used to serve image files (called `S3BUCKET`).

### Build
Run `docker-compose build` to build the images. This may take some time.

### Start 
Run `docker-compose up` to start the containers.

### Setup for live instance
- The setup is similar to running locally, but this time, the documentation assumes you are using AWS to host your instance
- The hardware requirements and gaining access to make this instance server is described below

By Kevin Gill, Joshua Rodriguez, and Robert Tapella
#### Architecture Diagram
<img width="569" alt="Screen Shot 2022-01-10 at 20 34 15" src="https://user-images.githubusercontent.com/72052911/148881690-2d33ea49-7ac7-4ef9-8672-f06d5d4915a1.png">

#### Infrastructure Assumptions
- EC2 Instance (r4.4xlarge is baseline)
- AMI image to copy to the EC2 instance
- A domain name that can be pointed at the working web server
- Web server configuration instructions
- AWS Postgres cluster with at least one writer and one reader (db.r4.2xlarge for each baseline; may scale reader up to 15 instances). At present, each database instance will contain between two to three gigabytes of data. [~2 GB of data]
Database export that will be imported into the Postgres database
- S3 bucket [~4GB of data]
- Some way to copy the images and GeoJSON files from the ingestion S3 bucket to the operational one

###### EC2
The primary host is a stock Amazon Linux server running dockerized containers housing the front-end, back-end, and static resource hosting  services. A standardized AMI has been created to ease the setup and all build resources are provided. The Docker containers are run via docker-compose. Internally to the containers, the front-end and static resources are hosted on an Nginx HTTP server.


    - Single r4.4xlarge Instance
    - Public static IP
    - AMI: ‘MSF Apache and Backend - 2019-11-06’
    - Security Group allows incoming 80/443/22 (HTTP, HTTPS, SSH)
    - SSH Key Pair
    - Ability to establish SSHFS virtual mount to Coral SDS system (if performing ingestion)

###### RDS
Database is a clustered (load-balanced) instance of Amazon Aurora PostgreSQL 9.6.12. It is optimized for more performant reads (given scarcity of writes). Firewall rules should allow access from EC2 instance (backend), and owners network (for ingestion access). Backend will be configured to access cluster head endpoint rather than any individual nodes. Ingestion should be configured to write to Writer node specifically. 

    - At minimum: 
    Single db.r4.2xlarge 
    - Preferred: 
    Cluster w/ single db.r4.2xlarge (Writer), 
    and 1-n (autoscaling) db.r4.2xlarge (Reader)
    DB: Aurora PostgreSQL 9.6.12
    Extension: PostGIS
    Matching subnet to EC2 instance, allowing inbound port 5432

###### S3 
The S3 bucket should be publicly accessible (read-only) with configuration added to allow for any-origin CORS. Both the backend and client browsers will access the bucket via HTTP. Ingestion processed access the bucket via AWS S3 API (boto3).

    Bucket w/ Cross Origin HTTP (CORS) configured

######  Steps to obtain and start up the MSF portal
- Provide Account ID, e.g. 233488530198. We will provide permissions to the AMI image.
- Select Launch Instance
- Select "My AMIs" then check "Shared with me"
- Click "Select" next to "MSF Portal - Dockerized"
- Select an instance size.
- Click "Next: Configure Instance Details"
- Select settings relevant to your organization
- Click "Next: Add Storage"
- Click "Next: Add Tags"
- Click "Next: Configure Security Group"
- Add rules allowing HTTP and HTTPS
- Click "Review and Launch"
- Click "Launch"
- Assign a key-pair
- Click "Launch Instances"
- Click "View Instances"
- Wait for instance to be ready
- Log into the instance using a variation of
`$ sh -i mykeypair.pem ec2-user@ipaddress`
- Clone this Git repository. Then run the steps for `Setup (local instance)`

### FAQ
#### How do I edit the landing page?
This site uses a landing page by default. To edit its contents, see `src/components/LandingPage/LandingPage.js`
#### How do I edit the Help documents?
This site uses .md files to populate its `Help` container. To edit its contents, see all .md files under `default-data/msf-data/help/`
#### How do I ingest new data into the portal?
Please see the `msf-flow` and `msf-ingestion` repositories, which are separate from this one (they are also NOT submodules). Data ingestion is a separate process from running the web app itself.
