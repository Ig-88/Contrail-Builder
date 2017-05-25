Copyright 2017 Juniper Networks, Inc.  All rights reserved.                   
Licensed under the Juniper Networks Script Software LIcense (the "License").  
You may not use this script file except in compliance with the License,       
which is located at                                                             http://www.juniper.net/support/legal/scriptlicense/                           
Unless required by applicable law or otherwise agree to in writing by         
the parties, software distributed u nder the License is distributed on        
an "AS IS" BASIS, WITHOUT WARANTIES OR CONDITIONS OF ANY KIND, either        
express or implied.                                           

# Contrail-Builder
Scripting and tools to automatically install a Contrail All-In-One (Contrail-AIO)
VM on an ESXi server.

This is a packaged tool to help simplify install a Contrail-AIO VM.
It basically proxies the contrail install through the Contrai-Builder VM.  This
allows for less customization (and failure) and more automation (and success).
If you want more customization and/or a more permanent option, you may be
better suited manually installing.  

http://www.juniper.net/us/en/products-services/sdn/contrail/contrail-networking/

Tools used on Contrail-Builder:
- Ansible
- Packer
- dnsmasq
- NTP server
- IPtables
***
What **YOU** do:
1.  Provide an ESXi server or vSphere environment version 5 or above
2.  Download and import the "Contrail-Build.ova" into environment
3.  Provide the builder vm with 2 virtual interfaces
  A.  An interface on your local network with Internet access
  B.  An interface on a separate, non-routed network available only to the builder and Contrail VM.  
      (you may have to create a new port-group on the vSwitch.)
4.  Provide only the build VM (Contrail-Builder) an IP address/Subnet Mask/Gateway.  NTP and DNS are provided by the builder VM.

What **IT** does:
1.  Creates an AIO Contrail VM
  A.  Downloads the Ubuntu ISO
  B.  Builds the host VM with Packer
  C.  Installs and configures OpenStack and Contrail with Ansible
2.  Provides an NTP server and NAT forwarding/routing to the Contrail VM with IPtables
3.  Provides GUI access to Contrail and OpenStack THROUGH the Contrail-Builder VM with IPtables
4.  Setups up Contrail quickly and easily to provide basic look and feel for Contrail and OpenStack

#### Caveats and known issues
- The process and build isn't perfect
- Some errors presented during Ansible playbook(s)
- Install time can vary based upon server(s) resources 30 minutes - 1 hour
- package management (apt-get) on the Contrail VM will be broken after install
- exclusive to Ubuntu 14.04.5 LTS and Contrail v3.2.2 and VMware ESXi hypervisor(s)
- The `fab contrail_setup` command may not finish completely.  This is
discovered when issuing the `contrail-status` command and show something like
"*vRouter is NOT PRESENT*" or other failure.
- Packer does support vCenter/vSphere environments
- The builder VM will build 1 Contrail-AIO.
- If the initial build fails to install Contrail correctly **BUT** the packer build says that it completes and powered down the VM you can do 1 of 2 things:

  * manually complete the install
  * Reboot VM and rerun the Packer script.
    * Rebooting is necessary because the build manipulates IPtables on the Contrail-Builder VM via the `Contrail-post` ansible playbook to complete the install.  Rebooting will remove the IPtables NAT entries that prevent the installation of the necessary packages.
         

- In order to make this work, some information about your environment needs to be added/updated in the json config file (Contrail-AIO.json).  


* OpenStack = http://IP_address_YOU_provided_in_Step_4/horizon  
IE http://10.10.10.10./horizon

* Contrail = http://IP_address_YOU_provided_in_Step_4:8143      
IE https://10.10.10.10:8143
