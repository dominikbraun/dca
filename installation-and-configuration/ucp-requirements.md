## UCP requirements

You can install UCP on-premises or on the cloud. Be sure your infrastructure meets the following requirements.

### Hardware and software requirements

Common requirements for on-premises installation or cloud installation:
* Docker EE 17.06 or higher
* Linux kernel version 3.10 or higher
* A static IP address

**Minimum requirements**
* 8GB of RAM for manager nodes or nodes running DTR
* 4GB or RAM for worker nodes
* 3GB of free disk space

**Recommended production environments**
* 16GB of RAM for manager nodes or nodes running DTR
* 4 vCPUs for manager nodes or nodes running DTR
* 25-100GB of free disk space

Windows container images are typically larger than Linux ones and for that reason, you should consider provisioning more
local storage for Windows nodes and for DTR setups that will store Windows container images.

When planning for host storage, workflows based around `docker pull` through UCP will result in higher storage
requirements on manager nodes, since `docker pull` through UCP results in the image being pulled on all nodes.

> These requirements assume that manager nodes don't run regular workloads.

### Time synchronization

In distributed systems like Docker UCP, time synchronization is critical to ensure proper operation. As a best practice
to ensure consistency between the engines in a UCP swarm, all engines should regularly synchronize with a Network Time
Protocol (NTP) server. If a server's clock is skewed, unexpected behavior may cause poor performance or even failures.

### Compatibility and maintenance lifecycle

Docker EE is a software subscription that includes three products:
* Docker Engine with enterprise-grade support
* Docker Trusted Registry
* Docker Universal Control Plane

### See also
* [UCP System requirements](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/install/system-requirements/#hardware-and-software-requirements)
* [Compatibility Matrix](https://success.docker.com/Policies/Compatibility_Matrix)
* [Maintenance Lifecycle](https://success.docker.com/Policies/Maintenance_Lifecycle)