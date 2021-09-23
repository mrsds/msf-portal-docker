# msf-portal-docker

### Setup

#### Initialize the submodules
Run `git submodule init`, then `git submodule update` to update the submodules.

#### Set Variables
Start by setting the proper variables. Edit the file `/msf-ui/src/config.js` and the file `msf-be/src/msfbe/config.ini` on the line that starts with `s3.proxyurl=` to point to the correct URL. This should be the base URL of the site. **NOTE!** Leave the `/server` part at the end of the URL, as that maps to the path that's proxied to `msf-be`.

#### Save certs
In the `/certs` folder, place the `methane.jpl.nasa.gov.crt` and `methane.jpl.nasa.gov.key` files.

#### Configure the environment if necessary.
The `env.conf` file has envvars that are sent to the `msf-be` instance.

### Build
Run `docker-compose build` to build the images.

### Start 
Run `docker-compose up` to start the containers.
