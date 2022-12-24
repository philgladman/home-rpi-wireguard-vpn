# home-rpi-wireguard-vpn
Wireguard VPN on raspberry pi k3s cluster

## Prereq Steps
### Raspberry Pi Setup
- Please follow this link for instructions on [How to install and setup K3s Cluster on raspberry pi](https://github.com/philgladman/home-rpi-k3s-cluster.git)
- This link will show you how to;
  - Prepare your raspberry pi for kubernetes
  - Spin up K3s
  - Install MetalLb
  - Install Nginx Ingress
- If you already have a Raspberry pi configured with K3s, MetalLb, and Nginx ingress, please move on to [Step 1.)](README.md#step-1---setup-external-drive-for-nfs-server)


## Step 1.) - Configure DYNU DNS
- Go to https://www.dynu.com/ and create an account
- Once logged in with your new accout go to the [Control Plane](https://www.dynu.com/en-US/ControlPanel), and click on `DDNS Services`.
- Click `Add`, and use Option 1 to create a custom domain to point to your public IP. For this we will use `this-is-a-test.mywire.org`.
- Click add, and then on the next page click save.
- Now we have a url that points to our Public IP. However, since you more than likely have a dynamic public IP, as soon as your Public Ip changes, this URL will no longer point to your public IP.

## Step 2.) - Confiugre script to update IP Address
- Run the following on the master node
- create a directory for DYNU `sudo mkdir ~/DYNU`
- create our updateIP.sh file `sudo touch ~/DYNU/updateIP.sh`
- Change updateIP.sh perms and ownership `sudo chown root:root ~/DYNU/updateIP.sh && sudo chmod 700 ~/DYNU/updateIP.sh`
- create a MD5 hash of your DYNU password with this `echo -n "your-password" | md5sum`
- add the following to our `sudo vim ~/DYNU/updateIP.sh` script.
- `echo url="https://api.dynu.com/nic/update?username=<your-DYNU-username>&password=<your-DYNU-password-hash>" | curl -k -o /home/ubuntu/DYNU/dynu.log -K -` Replacing the values with your DYNU username and password hash.
- Now that we have our script configured, lets create a cronjob that runs the script every 5 minutes `echo "*/5 * * * * /home/pi/DYNU/updateIP.sh" >> /var/spool/cron/crontabs/root`. You will need to run this as root, `sudo su`.
- Now that our Public Ip is getting updated every 5 minutes, we can move forward with installing Wireguard VPN.

## Step 2.) - Configure and install Wireguard VPN
- 

