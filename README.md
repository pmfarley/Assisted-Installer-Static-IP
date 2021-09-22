# Assisted-Installer-Static-IP
Using the OCP Assisted Installer with API to create a Static IP based ISO.


**STEP 1: Open the Assisted Installer Web UI:**
  https://console.redhat.com/openshift/assisted-installer/clusters/


**STEP 2: Cick on Create Cluster:**
  ![image](https://user-images.githubusercontent.com/48925593/134395134-1665ad54-7c20-4251-a436-9efedb0fe764.png)


**STEP 3: Specify the Cluster name, Base domain, and OpenShift version, and click Next:**
  ![image](https://user-images.githubusercontent.com/48925593/134395722-86f875ad-016a-4d2c-92d0-222dc1a2b091.png)


**STEP 4: Gather the Cluster ID from the URL:**
  ![image](https://user-images.githubusercontent.com/48925593/134409953-25f6086c-a016-4de4-94cf-10d79e8d5d76.png)

**    a. The url speciffied is: **
https://console.redhat.com/openshift/assisted-installer/clusters/975ec989-3264-4bb2-adfc-7846dcc7f29f

**    b. Assign this into the variable CLUSTER_ID:**

  ```bash
  export CLUSTER_ID="975ec989-3264-4bb2-adfc-7846dcc7f29f"
  ```

**STEP 5: LOAD TOKEN from the OpenShift Cluster Manager site: **
  https://console.redhat.com/openshift/token

**  a. Click on Load token.**

    ![image](https://user-images.githubusercontent.com/48925593/134413755-7abed6eb-4b41-4263-ad1d-8c455a9881b0.png)

**  b. Click on the Copy to Clipboard icon to the right of the token.**

    ![image](https://user-images.githubusercontent.com/48925593/134415323-59db0811-bea2-42f1-a859-2c2b1c4ffa2a.png)

**  c. Assign this into the variable OFFLINE_ACCESS_TOKEN:**

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

STEP 6: 
