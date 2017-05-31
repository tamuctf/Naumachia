# Naumachia

### A multi-tenant network sandbox for security challenges

The ambition of this project is to develop a system for deploying network exploit challenges for fun and non-profit. The main target is providing fun and challenges excersizes for a CTF, with the secondary goal of being used in a classroom setting. 

It provides challengers with simulated layer two (LAN) access to their own instance of a network and target machines defined by the host. When a user logs in a cluster is brought online just for them, and when they log out it is torn down, saving any persistant data, until they come back.

The two primary components of this project are OpenVPN, which provide authentication and layer two access accross the Internet, and Docker Compose, which manages the challenege environements and provides the language through which these environments are defined. Python code exists to couple these systems; coordinating the cluster creation/teardown, creation of user credentials, orchistration of platform networking, and various other functions which turn these powerful tools into Naumachia. 

## Setup

#### Initial Setup
1. Install [Docker](https://docs.docker.com/engine/installation/) and [Docker Compose](https://docs.docker.com/compose/install/)
2. Clone Naumachia onto a Linux server (developed and tested with Ubuntu)
3. Install requirements.txt into Python3 (i.e. `pip3 install -r requirements.txt`)

#### On Boot
For lack of a better method there are two steps that will need to be completed on intial installation and every time Naumachia will be run after reboot. It is my intention to obviate the need for these steps as development continues.
1. Disable bridge-nf enforcement of iptables rules by running `disable-bridge-nf-iptables.sh`. This is needed to allow unrestricted access within the sandbox network Naumachia places the user, which Docker restricts to increase security within its virtual networks. *This does not effect layer 3 restrictions imposed by iptables*
2. Run an arbitrary docker container on the host network (e.g. `docker run --rm --net=host alpine /bin/true`) This create a link in the `/var/run/docker/netns` folder called `default` which allows access to the host network namespace. This is then added to the cluster-manager container to allow it to modify the bridges generated by Docker at runtime.

#### On Challenge Creation
1. Write a [`docker-compose.yml` template](https://docs.docker.com/compose/compose-file/) and put it and any associated files in directory within the `challenges` directory. See `challenges/example` for some guidence
2. Create a directory within `openvpn/config` for your tunnel configurations. You can copy `openvpn/config/example` and change anything in `ovpn_env.sh` or `openvpn.conf` as you see fit.
3. Initialise the PKI following directions from [kylemanna/openvpn](https://github.com/kylemanna/docker-openvpn), with the folder you just created as the openvpn config folder.
4. Modify `config.yml` to include your challenge
5. Run `./render-docker-compose.py` to generate the `docker-compose.yml` file from a Jinja2 template

#### Run it!
To run Naumachia simply bring up the enviroment with [Docker Compose CLI](https://docs.docker.com/compose/reference/overview/) (e.g. `docker-compose up -d`)
