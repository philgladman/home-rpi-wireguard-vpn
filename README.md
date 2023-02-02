# home-rpi-wireguard-vpn
Wireguard VPN on raspberry pi k3s cluster

## Prereq Steps
- K3s Cluster running on Raspberry Pi(s)
- MetalLB
- Nginx Ingress
- NFS Server install on K3s Cluster

### Raspberry Pi Setup
- Please follow this link for instructions on [How to install and setup K3s Cluster on raspberry pi](https://github.com/philgladman/home-rpi-k3s-cluster.git)
- This link will show you how to;
  - Prepare your raspberry pi for kubernetes
  - Spin up K3s
  - Install MetalLb
  - Install Nginx Ingress
### NFS Setup
- Please follow this link for instructions on [How to install and setup a NFS Server on raspberry pi K3s Cluster](https://github.com/philgladman/home-rpi-NFS.git)

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

## Step 3.) - Configure and install Wireguard VPN
- clone this repo `git clone https://github.com/philgman1121/home-rpi-wireguard-vpn.git`
- cd into the repo `cd home-rpi-wireguard-vpn`
- edit the `kustomize/wireguard-cm.yaml` to have your values
- set the `TZ` variable to have your Timezone.
- set the `SERVERURL` to have your DYNU url.
- for this demo we will create 2 peers `philiphone` and `philmackbook`. Feel free to change the names, or add more/less. The peer name must only contain letters and numbers.
- `PEERDNS` is set to `auto`. If you have a pihole running, you can set this `PEERDNS` to the clusterIP of your pihole svc.
- `INTERNAL_SUBNET` is the subnet on the vpn.
- After all the values have been customized, deploy wireguard with `kubectl apply -k kustomize/.`
- On your router, you will need to port foward port 51820 UDP to the ip of the wireguard svc(wireguard svc ip = `kubectl get svc -n wireguard wireguard -o jsonpath='{.status.loadBalancer.ingress[].ip}'`)

## Step 4.) - Retrieve Wireguard VPN Configuration
- Once the wireguard pod is up and running, run the following command `kubectl logs -n wireguard $(kubectl get pods -n wireguard -o jsonpath='{.items[].metadata.name}')`
- This will output the logs of the wireguard container, which will have a qr code for each peer. 
- Download the wireguard app on your phone. 
- Once downloaded, click the plus sign to add a new wireguard tunnel
- click `Create from QR code`
- scan the QR code from earlier.
- Now you can turn your new VPN on and test.
