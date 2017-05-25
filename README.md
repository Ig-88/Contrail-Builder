#### Contrail-Builder
Scripting and tools to automatically install a Contrail All-In-One VM on an ESXi
server.

This is a packaged tool to help simply install a Contrail AIO(All-In-One VM.
It basically proxies the contrail install through the Contrai-Builder VM.  This
allows for less customization (and failure) and more automation (and success).
If you want more customization and/or a more permanent option, you may be
better suited manually installing.  

http://www.juniper.net/us/en/products-services/sdn/contrail/contrail-networking/

Tools used on Contrail-Builder:
Ansible
Packer
dnsmasq
NTP server
IPtables

What # YOU do:
1.  Provide an ESXi server or vSphere environment version 5 or above
2.  Download and import the "Contrail-Build.ova" into environment
3.  Provide the builder vm with 2 virtual interfaces
  A.  An interface on your local network with Internet access
  B.  An interface on a separate, non-routed network available only to the builder and Contrail VM.  
      (you may have to create a new port-group on the vSwitch.)
4.  Provide the build VM (only) an IP address/Subnet Mask/Gateway.  NTP and DNS are provided

What IT does:
1.  Creates an AIO Contrail VM
  A.  Downloads the Ubuntu ISO
  B.  Builds the host VM with Packer
  C.  Installs and configures OpenStack and Contrail with Ansible
2.  Provides an NTP server and NAT forwarding/routing to the Contrail VM with IPtables
3.  Provides GUI access to Contrail and OpenStack THROUGH the Contrail-Builder VM with IPtables
4.  Setups up Contrail quickly and easily to provide basic look and feel for Contrail and OpenStack

Caveats and known issues
- Some errors presented during Ansible playbook(s)
- Install time can vary based upon server(s) resources 25 minutes - 1 hour
- package management (apt-get) on the Contrail VM will be broken after install
- exclusive to Ubuntu 14.04.5 LTS and Contrail v3.2.2 and VMware ESXi hypervisor(s)
- The "fab contrail_setup" command may not finish completely.  This is discovered when issuing the          "contrail-status" command     and show something like this "vRouter is NOT PRESENT" or other failure.
- Packer does support vCenter/vSphere environments

In order to make this work, some information about your environment needs to be
added/updated in the json config file.


GUI access
OpenStack = http://IP_address_YOU_provided_in_Step_4/horizon  IE http://10.10.10.10./horizon
Contrail = http://IP_address_YOU_provided_in_Step_4:8143      IE https://10.10.10.10:8143

User Crendentials - CLI
contrail/contrail123
root/contrail123

User Credentials - GUI
admin/contrail123
