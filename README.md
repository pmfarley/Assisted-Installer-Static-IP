# Assisted-Installer-Static-IP
Using the OCP Assisted Installer with API to create a Static IP based ISO


**STEP 1: Open the Assisted Installer Web UI:**
https://console.redhat.com/openshift/assisted-installer/clusters/


**STEP 2: Cick on Create Cluster:**
![image](https://user-images.githubusercontent.com/48925593/134395134-1665ad54-7c20-4251-a436-9efedb0fe764.png)


**STEP 2: Specify the Cluster name, Base domain, and OpenShift version, and click Next:**
![image](https://user-images.githubusercontent.com/48925593/134395722-86f875ad-016a-4d2c-92d0-222dc1a2b091.png)


**STEP 3: Gather the Cluster ID from the URL:**

The url speciffied is: 
https://console.redhat.com/openshift/assisted-installer/clusters/975ec989-3264-4bb2-adfc-7846dcc7f29f

export CLUSTER_ID="975ec989-3264-4bb2-adfc-7846dcc7f29f"


**STEP 4: LOAD TOKEN from **

