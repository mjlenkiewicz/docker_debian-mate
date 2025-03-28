This container provides a secure environment for testing purposes.While containers aren't
traditionally designed for persistent workspaces, proper configuration can make them highly
effective for this use case.

IMPORTANT: You must have installed docker engine (Linux) or docker desktop (Windows) !!!

for Linux see ...
https://docs.docker.com/engine/install/

for Windows see ...
https://docs.docker.com/desktop/setup/install/windows-install/

Then, with the files already on your PC and the zip file uncompressed in a folder with the
same name (assets), run the following commands ...

1) create a network for using with the container
# on Linux
> sudo docker network create -d bridge --subnet 192.168.5.0/24 --gateway 192.168.5.1 mydockernet
# on Windows (same without "sudo" command)
2) build the container image usign the dockerfile provided
# on Linux
sudo docker build -f Dockerfile.mate -t debian-mate:dev .
# on Windows (same without "sudo" command)
3) run the container (you must see the bash when it started, then read User Credentials and Access to MATE GUI sections)
# on Linux
sudo docker run --rm -ti -p 3390:3389 --net=mydockernet --ip="192.168.5.5" --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=SYS_ADMIN --cap-add=CAP_DAC_READ_SEARCH -h debian debian-mate:dev
# on Windows (same without "sudo" command)

Command "docker run" options reference:
--rm
Automatically removes the container when it stops or exits. Useful for temporary testing where
persistence isn’t needed.

-ti Combines two options:
-t: Assigns a pseudo-TTY for an interactive interface.
-i: Keeps the standard input open for container interaction.

-p 3390:3389
Maps port 3389 inside the container to port 3390 on the host, enabling external access to services
running in the container.

--net=mydockernet
Connects the container to a custom Docker network named mydockernet, allowing communication with
other containers on the same network.

--ip="192.168.5.5"
Assigns a static IP address (192.168.5.5) within the specified network (mydockernet). Requires a
network driver that supports static IPs (e.g., bridge).

--cap-add
Adds Linux capabilities to the container:
a» NET_ADMIN: Allows modifying network interfaces.
b» NET_RAW: Enables advanced network operations (e.g., raw sockets).
c» SYS_ADMIN: Grants system-level access (e.g., mounting devices).
d» CAP_DAC_READ_SEARCH: Permits reading files/directories even if not explicitly permitted.

a/b needed for wireshark, tcpdump, etc
c/d needed for mounting volumen & using samba

-h debian
Sets the container’s hostname to debian, visible in commands like hostname.

## Included Tools
The container features network debugging and simulation tools such as:
* visual studio code
* miniconda3 & python 3.9 virtal environment (py39env)
* nodejs & npm
* git & all sub-packages
* network file sharing tools
    - samba (for connecting to Windows network shared directories through dir. explorer «CAJA»)
    - filezilla client
* networking tools:
    - ethtool
    - net-tools
    - iputils-ping
    - traceroute
    - tcpdump
    - nmap
    - Wireshark (is not installed; if you would like to install it, uncomment the corresponding
                lines)

## Software Management
If you do not require the included software, comment out or remove the corresponding entries
in the Dockerfile, and/or you can extend functionality by adding custom software modifying the
dockerfile or using a runtime installation, but remember that changes made during runtime (via
GUI/terminal) will be ephemeral unless you commit the modified container to a new image or update
the original Dockerfile and rebuild.


## User Credentials
1)
USER = root
PASS = root9355
2)
USER = test
PASS = test9455

## Access to MATE GUI (only via remote desktop connection)
Open an RDP connection to localhost:3390 and enter the user credentials. It is recommended
to access using the 'test' user and use the sudo command for running privileged operations.

To modify passwords, edit the Dockerfile and regenerate hashed credentials using Cryptool's
OpenSSL tool (www.cryptool.org/en/cto/openssl).

## Post-Deployment Workflow
a) Temporary Use: No further action required.
b) Persistent Work: For development/debugging, preserve data by:
    - Creating a derived image with critical files
    - Using Docker volumes for persistent storage
    - Implementing regular backups
