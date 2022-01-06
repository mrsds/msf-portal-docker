# msf-portal-docker
README by Josh Rodriguez and Kathryn Yu

### Setup

### Prerequisites
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

### FAQ: How do I ingest new data into the portal?
Please see the `msf-ingestion` repository, which is separate from this one and is NOT one of this web app's submodules. Data ingestion is a separate process from running the web app itself.
