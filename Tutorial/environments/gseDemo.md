# GSE Demo Central
GSE provides the capabilities to set up API Platform for short-term demos.  You can also use this option if you are holding a workshop and wish to train customers and partners.

Use http://demo.oracle.com.  Select **UCM with OCI Focus**.  That will set up a cloud domain, which you can provision API Platform CS management Service.

You can also provision an IaaS Compute to install the gateway.

Log into the **myServices** dashboard, select **Autonomous API Platform Cloud Service** and create an instance.  Do not worry about prerequisites!

>Note: Your login will automatically be the API Platform administrator.  You can give other users **Appliction Roles** by selecting the **Users**, the user you wish to update, and select roles.  Find the service that has your instance name.  For example, if your instance is **APIDemo**, look for **APICSAUTO_APIDemo** and add the role(s) desired for the user.

You can log into your management portal either by navigating to **Autonomous API Platform Cloud Service** under **myServces**, choosing **Open Service Console**, then choosing **Access API Platform Service Instance** for your particular instance in the **Instances** list.

You can also navigate directly to the management console by pointing your browser to `https://<instance name>-<domain name>.apiplatform.ocp.oraclecloud.com/apiplatform`

The developer portal is similar, just replace **apiplatform** with **developers**