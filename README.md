**MAP SCION**

This setup uses vagrant and virtualbox to create a freestanding scion network as described on https://docs.scion.org/en/latest/tutorials/deploy.html

For this setup, install the following requirements:
- Install VirtualBox v7.0
- Install vagrant 2.4.3

To run the setup, clone this repository and run 
```console
vagrant up
```
in your console.

ATTENTION:
After setup, /etc/hosts/ needs to be modified by deleting the double alias for scion0i binding to a loopback address
e.g. "127.0.2.1 scion01 scion01" needs to be removed
TODO: Fix this automatic binding
