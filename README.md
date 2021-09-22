# Assisted-Installer-Static-IP
Using the OCP Assisted Installer with API to create a Static IP based ISO.


**STEP 1. Open the Assisted Installer Web UI:**
  https://console.redhat.com/openshift/assisted-installer/clusters/


**STEP 2. Cick on Create Cluster:**
  ![image](https://user-images.githubusercontent.com/48925593/134395134-1665ad54-7c20-4251-a436-9efedb0fe764.png)


**STEP 3. Specify the Cluster name, Base domain, and OpenShift version, and click Next:**
  ![image](https://user-images.githubusercontent.com/48925593/134395722-86f875ad-016a-4d2c-92d0-222dc1a2b091.png)


**STEP 4. Gather the Cluster ID from the URL:**
  ![image](https://user-images.githubusercontent.com/48925593/134409953-25f6086c-a016-4de4-94cf-10d79e8d5d76.png)

**a. The url speciffied is:** 
https://console.redhat.com/openshift/assisted-installer/clusters/975ec989-3264-4bb2-adfc-7846dcc7f29f

**b. Assign this into the variable CLUSTER_ID:**
  ```bash
  export CLUSTER_ID="975ec989-3264-4bb2-adfc-7846dcc7f29f"
  ```

**STEP 5. LOAD TOKEN from the OpenShift Cluster Manager site:**
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

**STEP 6. ASSIGN OTHER CLUSTER PROPERTIES:**

**a. Assign the following parameters as variables. Modfy these for your environment.**

   ```bash
   export ASSISTED_SERVICE_API="api.openshift.com"
   export CLUSTER_VERSION="4.6"                                   # OpenShift version    
   export CLUSTER_IMAGE="quay.io/openshift-release-dev/ocp-release:4.6.16-x86_64"
   export CLUSTER_NAME="waiops"                                   # OpenShift cluster name    
   export CLUSTER_DOMAIN="redhat.local"                           # Domain name where my cluster will be deployed 
   export CLUSTER_NET_TYPE="OpenShiftSDN"                         # Set the Network type to deploy with OpenShift
   ```
   
**STEP 7. REFRESH OFFLINE TOKEN:**
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

**STEP 8. RETRIEVE CLUSTER CONFIG with API:**

      ```bash
      curl -s -X GET --header "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" "https://$ASSISTED_SERVICE_API/api/assisted-install/v1/clusters/$CLUSTER_ID/install-config"|jq -r
      ```
      
      SAMPLE OUTPUT:
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
      
