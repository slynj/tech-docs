# 5G Core - Building Open5GS

[Quickstart](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

This guide will walk you through installing Open5Gs in Ubuntu environment. Please check the official document for other environments as you might need to build from source.

## 1. Getting MongoDB

Import the public key used by the package management system.

```bash
sudo apt update
sudo apt install gnupg
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor
```

Create the list file /etc/apt/sources.list.d/mongodb-org-6.0.list for your version of Ubuntu.

On ubuntu 22.04 (Jammy):

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

## 2. Installing Open5GS

Install Open5GS using `apt`

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

Verify that it’s running

```bash
ps aux | grep open5gs
```

![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image.png)

## 3. Install the WebUI of Open5GS

The WebUI allows you to interactively edit subscriber data. 

Install `Node.js`

```bash
 # Download and import the Nodesource GPG key
 sudo apt update
 sudo apt install -y ca-certificates curl gnupg
 sudo mkdir -p /etc/apt/keyrings
 curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

 # Create deb repository
 NODE_MAJOR=20
 echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

 # Run Update and Install
 sudo apt update
 sudo apt install nodejs -y
```

Install WebUI

```bash
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

Now when you goto `http://localhost:9999`  you should be able the see the graphical UI like below.

Start WebUI

```bash
sudo systemctl start open5gs-webui
```

*OPTIONALLY* If you want to access the webUI from another machine, you can modify the `index.js` file to listen to `0.0.0.0`.

1. Locate where the `index.js` is at. 
    
    ```bash
    find / -name "open5gs" -type d 2>/dev/null
    ```
    
    ![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%201.png)
    
    Check the directories listed and find the `server` directory which is where the js file is in. Typically located in `/usr/lib/node_modules/open5gs/server/index.js`
    
2. Modify the `listen` call to bind to `0.0.0.0`'
    
    ```bash
    sudo nano /usr/lib/node_modules/open5gs/server/index.js
    ```
    
    ![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%202.png)
    
3. Optionally test with `curl`
    
    ```bash
    curl 192.168.1.6:9999
    ```
    
    ![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%203.png)
    
4. Connect to it on the machine. For me it was `http:://192.168.1.6:9999`

## 4. Configuring Open5GS

Now that you’ve installed Open5GS, you need to modify the configuration and set up your network for proper functionality. Here’s a brief overview:

1. Modify configuration files
    - Update the `*.yaml` files for each component in the `/etc/open5gs/` directory
    - Configure IP addresses, PLMN (Public Land Mobile Network) IDs, and other settings to match your deployment. Note that Open5GS comes ready to run on one machine, only modify IP addresses if your UE & gNB is on a different machine.
    - You will need to modify the PLMN in your NRF and AMF config, and in case of AMF, further modify the TAC information.
    - If you are aiming to connect an external gNB to your core, you will also need to change the NGAP bind address of the AMF **and** the GTPU bind address of the UPF. If you are running an gNB stack locally, you will not need to make these changes.
2. Add Subscriber Data
    - Enter your UE details like IMSI, keys, and APNs using the WebUI (or CLI)

### nrf.yaml

Modify `/etc/open5gs/nrf.yaml` to set the PLMN ID.

```yaml
nrf:
  serving:  # 5G roaming requires PLMN in NRF
    - plmn_id:
        mcc: 001
        mnc: 01
```

### amf.yaml

Modify `/etc/open5gs/amf.yaml` to set the NGAP IP address, PLMN ID, TAC and NSSAI.

```yaml
ngap:
    server:
      - address: 127.0.0.5
  metrics:
    server:
      - address: 127.0.0.5
        port: 9090
  guami:
    - plmn_id:
        mcc: 001
        mnc: 01
      amf_id:
        region: 2
        set: 1
  tai:
    - plmn_id:
        mcc: 001
        mnc: 01
      tac: 1
  plmn_support:
    - plmn_id:
        mcc: 001
        mnc: 01
      s_nssai:
        - sst: 1

```

### upf.yaml

Modify `/etc/open5gs/upf.yaml` to set the GTP-U address.

```yaml
     gtpu:
       server:
        - address: 127.0.0.7
     session:
       - subnet: 10.45.0.1/16
       - subnet: 2001:db8:cafe::1/48
```

After configuring, restart the daemons.

```bash
sudo systemctl restart open5gs-nrfd
sudo systemctl restart open5gs-amfd
sudo systemctl restart open5gs-upfd
```

## 5. Registering Subscriber Information

Connect and login with admin account

```
ID: admin
PWD: 1423
```

![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%204.png)

Add subscriber information

1. Go to `subscriber` Menu
    
    ![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%205.png)
    
2. Click `+` button to add a new subscriber
    
    
3. Fill the IMSI, security context(K, OPc, AMF), and APN of the subscriber
    
    ![image.png](5G%20Core%20-%20Building%20Open5GS%2017e011ff45b98048a6aff1a15e6d60a5/image%206.png)
    
4. Click `SAVE` 

## 6. Uninstalling Open5GS and WebUI

Remove the Open5GS packages.

```yaml
sudo apt purge open5gs
sudo apt autoremove
```

Remove the logs as well.

```yaml
sudo rm -Rf /var/log/open5gs
```

Remove the WebUI.

```yaml
curl -fsSL https://open5gs.org/open5gs/assets/webui/uninstall | sudo -E bash -
```