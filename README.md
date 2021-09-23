# Assisted-Installer-Static-IP
Using the OCP Assisted Installer with API to create a Static IP based ISO.


**STEP 1. DOWNLOAD THE PULL SECRET FROM THE URL:**
https://console.redhat.com/openshift/install/pull-secret

**a. Click on Download pull secret, and save as filename pull-secret.txt in your current folder.**
![image](https://user-images.githubusercontent.com/48925593/134422760-dc8b8010-56d4-4061-89c1-6e78a73a52db.png)


**STEP 2. LOAD TOKEN FROM THE OPENSHIFT CLUSTER MANAGER SITE:**
  https://console.redhat.com/openshift/token

**a. Click on Load token.**

![image](https://user-images.githubusercontent.com/48925593/134417909-5af81a67-4648-4827-b1ff-e533cffdd595.png)


**b. Click on the Copy to Clipboard icon to the right of the token.**

![image](https://user-images.githubusercontent.com/48925593/134417785-206fb11c-dfe4-4753-9fc8-166b8ee6de39.png)


**c. Assign this into the variable OFFLINE_ACCESS_TOKEN:**

   ```bash
   export OFFLINE_ACCESS_TOKEN="<PASTE_TOKEN_HERE>"

   export TOKEN=$(curl \
   --silent \
   --data-urlencode "grant_type=refresh_token" \
   --data-urlencode "client_id=cloud-services" \
   --data-urlencode "refresh_token=${OFFLINE_ACCESS_TOKEN}" \
   https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token | \
   jq -r .access_token)
   ```


**STEP 3. ASSIGN THE OTHER CLUSTER PROPERTIES:**

**a. Assign the following parameters as variables. Modfy these for your environment.**

   ```bash
   export ASSISTED_SERVICE_API="api.openshift.com"
   export CLUSTER_VERSION="4.6"                                                   # OpenShift version    
   export CLUSTER_IMAGE="quay.io/openshift-release-dev/ocp-release:4.6.16-x86_64"
   export CLUSTER_NAME="waiops"                                                   # OpenShift cluster name    
   export CLUSTER_DOMAIN="redhat.local"                                           # Domain name where the cluster will be deployed 
   export CLUSTER_NET_TYPE="OpenShiftSDN"                                         # Set the Network type to deploy with OpenShift
   export PULL_SECRET=$(cat ./pull-secret.txt | jq -R .)                          # Loading the pull-secret into variable
   export CLUSTER_SSHKEY=$(cat ~/.ssh/id_rsa.pub)                                 # Loading the public key into variable
   ```
   
**STEP 4. GENERATE THE DEPLOYMENT.JSON FILE:**

   ```bash
   cat << EOF > ./deployment.json
   {
   "kind": "Cluster",
   "name": "$CLUSTER_NAME",
   "openshift_version": "$CLUSTER_VERSION",
   "ocp_release_image": "$CLUSTER_IMAGE",
   "base_dns_domain": "$CLUSTER_DOMAIN",
   "network_type": "$CLUSTER_NET_TYPE",
   "user_managed_networking": false,
   "vip_dhcp_allocation": false,
   "ssh_public_key": "$CLUSTER_SSHKEY",
   "pull_secret": $PULL_SECRET
   }
   EOF
   ```


**STEP 5. REFRESH THE OFFLINE TOKEN:**
(This may need to be performed periodically)


   ```bash
   export TOKEN=$(curl \
      --silent \
      --data-urlencode "grant_type=refresh_token" \
      --data-urlencode "client_id=cloud-services" \
      --data-urlencode "refresh_token=${OFFLINE_ACCESS_TOKEN}" \
      https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token | \
      jq -r .access_token)
   ```

**STEP 6. CREATE THE CLUSTER USING THE DEPLOYMENT.JSON FILE:**

**a. Create cluster.**
   ```bash
   export CLUSTER_ID=$( curl -s -X POST "https://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters" \
  -d @./deployment.json \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  | jq '.id' )
   ```
  
**b. Assign the variable CLUSTER_ID:**
  ```bash
  export CLUSTER_ID=$( sed -e 's/^"//' -e 's/"$//' <<<"$CLUSTER_ID")
  
  echo $CLUSTER_ID
  e85fc7d5-f274-4359-acc5-48044fc67132
   ```

**STEP 7. VERIFY/RETRIEVE THE CLUSTER CONFIG with API:**

  ```bash
      curl -s -X GET --header "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" \
      "https://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters/$CLUSTER_ID/install-config"|jq -r
   ```
      
SAMPLE OUTPUT:
  ```bash
      apiVersion: v1
      baseDomain: redhat.local
      networking:
        networkType: OpenShiftSDN
        clusterNetwork: []
        serviceNetwork: []
      metadata:
        name: waiops
      compute:
      - hyperthreading: Enabled
        name: worker
        replicas: 0
      controlPlane:
        hyperthreading: Enabled
        name: master
        replicas: 0
      platform:
        baremetal:
          provisioningNetwork: Unmanaged
          apiVIP: ""
          ingressVIP: ""
          hosts: []
        vsphere: null
      fips: false
      pullSecret: 'Your-Pull-Secret'
      sshKey: 'Your-SSH_KEY'
  ```
      
**STEP 8. CREATE THE NMSTATE YAML FILES:**

    Create a yaml file for each node in the cluster (master-0, master-1, master-2, worker-0, worker-1, worker-2).
    The master-0 file is shown below. 
    Replicate this for the other nodes, changing the IP addresses to match your environment.
    
    NOTE: vSphere virtual ethernet adapter device name shows up as "ens192".
    
  ```bash
cat << EOF > ./master-0.yaml 
dns-resolver:
  config:
    server:
    - 192.168.1.210
interfaces:
- ipv4:
    address:
    - ip: 192.168.2.100
      prefix-length: 24
    dhcp: false
    enabled: true
  name: ens192
  state: up
  type: ethernet
routes:
  config:
  - destination: 0.0.0.0/0
    next-hop-address: 192.168.2.1
    next-hop-interface: ens192
    table-id: 254
EOF
   ```
   
**STEP 9. GENERATE THE DISCOVERY ISO FILE USING THE NMSTATE FILES:**

  Gather the MAC addresses for all of your VMs/Baremetal nodes. 
  Edit the MAC addresses below for each of the nodes to match your environment.
  
```bash
DATA=$(mktemp)

jq -n --arg SSH_KEY "$CLUSTER_SSHKEY" \
--arg NMSTATE_YAML1 "$(cat ./master-0.yaml)" --arg NMSTATE_YAML2 "$(cat ./master-1.yaml)" \
--arg NMSTATE_YAML3 "$(cat ./master-2.yaml)" --arg NMSTATE_YAML4 "$(cat ./worker-0.yaml)" \
--arg NMSTATE_YAML5 "$(cat ./worker-1.yaml)" --arg NMSTATE_YAML6 "$(cat ./worker-2.yaml)" \
'{
  "ssh_public_key": $SSH_KEY,
  "image_type": "full-iso",
  "static_network_config": [
    {
      "network_yaml": $NMSTATE_YAML1,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:02:7b", "logical_nic_name": "ens192"}]
    },
    {
      "network_yaml": $NMSTATE_YAML2,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:ff:58", "logical_nic_name": "ens192"}]
    },
    {
      "network_yaml": $NMSTATE_YAML3,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:72:7d", "logical_nic_name": "ens192"}]
     },
    {
      "network_yaml": $NMSTATE_YAML4,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:d8:09", "logical_nic_name": "ens192"}]
    },
    {
      "network_yaml": $NMSTATE_YAML5,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:1e:92", "logical_nic_name": "ens192"}]
    },
    {
      "network_yaml": $NMSTATE_YAML6,
      "mac_interface_map": [{"mac_address": "00:50:56:b9:be:33", "logical_nic_name": "ens192"}]
     }
  ]
}' > $DATA


curl -X POST "https://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters/$CLUSTER_ID/downloads/image" \
  -H "Content-Type: application/json"  -H "Authorization: Bearer $TOKEN" -d @$DATA
```

**STEP 10. DOWNLOAD THE DISCOVERY ISO FILE:**
   ```bash
   curl -L "http://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters/$CLUSTER_ID/downloads/image" \
   -o ~/discovery-image-$CLUSTER_NAME.iso  -H "Authorization: Bearer $TOKEN"
  ```

**STEP 11. RETRIEVE THE AWS S3 DOWNLOAD URL (OPTIONAL):**
   ```bash
   curl -s -X GET "https://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters/$CLUSTER_ID" \
   -H "Authorization: Bearer $TOKEN"|jq .image_info
   ```
   
SAMPLE OUTPUT:
   ```bash
     "download_url": "https://s3.us-east-1.amazonaws.com/assisted-installer/discovery-image-....", 
     "expires_at": "2021-08-19T07:11:46.229Z"
   ```
**STEP 12. BOOT EACH OF YOUR VM/BAREMETAL WITH THE DISCOVERY ISO IMAGE.**

**STEP 13. OPEN THE ASSISTED INSTALLER WEB UI:**
  https://console.redhat.com/openshift/assisted-installer/clusters/

a. You will see a list of clusters, click on the name of the cluster. 
![image](https://user-images.githubusercontent.com/48925593/134430565-5640e3ca-7111-49b2-90f1-8813f65c5e5b.png)


**STEP 14. In Host discovery Tab, once all of your nodes appear in the list, click on Next.**

**STEP 15. In the Networking tab, enter the settings for the Cluster network CIDR, Host prefix, and Service network CIDR. **

From this tab, also enter the static IP addresses for the API VIP and the Ingress VIP. 

**STEP 16. Proceed to click on Next, and then on Install the cluster.**

