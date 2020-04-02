Geth Autossh proxy setup
Edmund Edgar, 2020-04-02

These are the settings for exposing a Geth node via a cloud host.

(Browser) -> [ HTTP RPC over HTTPS ] -> (Nginx on cloud host) -> [ HTTP RPC over SSH ] -> (Geth on box in cupboard)

The cloud host runs Nginx, with multiple hosts for different networks as subdomains using SNI, proxying to a dedicated port on localhost (we'll use 18545).
The local Geth host runs Geth with RPC exposed on localhost, and exposes it to the said port on localhost of the cloud host using a persistent SSH tunnel.

Motivation:
* We want to run a node and expose it over RPC for dapp users.
* We don't care if it falls over occasionally, we're set up to fall back on Infura but having our own node makes the experience faster.
* Due to disk requirements a node on a cloud host is a little bit expensive, but the physical disks are dead cheap.
* We don't like all the nodes to be on cloud hosts because decentralization.
* We have bandwidth to spare at home.
* We don't have a static IP at home.
* We don't really want to expose the home IP to the world via our dapp as cybercoin enthusiasts sometimes play silly DoS games.


A) The cloud host

1) Setup DNS to point to the cloud host:

+mainnet.socialminds.jp:188.166.83.139:86400
+ropsten.socialminds.jp:188.166.83.139:86400
+rinkeby.socialminds.jp:188.166.83.139:86400
+kovan.socialminds.jp:188.166.83.139:86400
+goerli.socialminds.jp:188.166.83.139:86400

2) Setup Nginx. Everything is proxied to the Geth port except the .acme directory, which we'll need for Let's Encrypt.

sudo apt-get install nginx

# sudo mkdir -p /var/www/mainnet/html/.acme
[repeat for other hosts]

3) Copy cloud_host/etc/nginx/sites-available/mainnet to the same location on the cloud host. Delete the SSL stuff. Then:

# sudo nginx -t
# sudo systemctl restart nginx

4) Install certbot from some website.

5) Run certbot 
# sudo certbot -d mainnet.socialminds.jp
[repeat for other hosts]

6) Make a user, we'll call it ethnode.
# sudo adduser ethnode


B) The local box, should have SSH > 150GB and HDD > 150GB (this will grow over time)

1) Make a user, also called ethnode.
# sudo adduser ethnode

2) Make a key, don't give it a passphrase
# sudo su - ethnode
# ssh-keygen

3) Copy the resulting id_rsa.pub to /home/ethnode/.ssh/authorized_keys on the cloud host.

4) Make sure you can ssh from the local node to the cloud host.
# sudo su - ethnode
# ssh mainnet.socialminds.jp
# [Y]
# exit

5) Mount the HDD as /data, or symlink /data to wherever it's mounted.

6) Set up a directory for Geth files that don't need a fast disk.
sudo mkdir /data/geth_ancient
sudo chown ethnode /data/geth_ancient

7) Install Geth from the Geth website and sync it up.
sudo su - ethnode
geth --datadir.ancient=/data/geth_ancient 

8) Set Geth to start automatically, exposing RPC locally:
Copy local_box/etc/systemd/system/geth.service to the same location on the local box, then:
# sudo systemctl daemon-reload
# sudo systemctl enable geth.service
# sudo systemctl start geth
Any errors will be in /var/log/syslog.

9) Install autossh
# sudo apt-get install autossh

10) Setup autossh to start automatically
Copy local_box/etc/systemd/system/autossh-mainnet.service to the same location on the local box, then:
# sudo systemctl daemon-reload
# sudo systemctl enable autossh-mainnet.service
# sudo systemctl start autossh-mainnet
Any errors will be in /var/log/syslog.

11) Reboot to make sure everything comes back up.

