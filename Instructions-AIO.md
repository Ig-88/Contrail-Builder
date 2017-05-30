# Instructions for installing Contrail-Builder and All-In-One VM
The Contrail-Builder VM acts as a the installer and proxy for any and all
connectivity to/from the Contrail-AIO VM.

Prep
1.  Obtain a routable/networked IP address (RFC1918) that can connect to the Internet to
    download updates, packages and applications.  This IP is for the Contrail-Builder VM
    and gets assigned to `eth0/vnic1`.

    **Note** `eth1/vnic2` is statically assigned for the build and setup in the
    next step.

2.  Create a new port-group on the ESXi server.  This port-group should be a
    non-routed port-group that will only be used to connect the Contrail-Builder and
    Contrail-AOI VMs to each other.
***


### Install
1.  Enable the "Guest IP Hack" on the ESXi server by connecting via SSH and running
    this command from the cli;
    `esxcli system settings advanced set -o /Net/GuestIPHack -i 1`
    More information is available here:
    https://www.packer.io/docs/builders/vmware-iso.html

2.  Allow VNC through the ESXi firewall:

  1. `chmod 644 /etc/vmware/firewall/service.xml`

  2. `chmod +t /etc/vmware/firewall/service.xml`

  3. Then append the range of ports you want open to the end of the file:
>
```
 <service id="1000">
   <id>packer-vnc</id>
   <rule id="0000">
     <direction>inbound</direction>
     <protocol>tcp</protocol>
     <porttype>dst</porttype>
     <port>
       <begin>5900</begin>
       <end>6000</end>
     </port>
    </rule>
    <enabled>true</enabled>
    <required>true</required>
  </service>
```

    4. Restore the Firewall permissions:
```
            chmod 444 /etc/vmware/firewall/service.xml
            esxcli network firewall refresh   
```

3.  Import the Contrail-Builder.ova file

4.  Power on the VM

5.  Log into the VM's console with `root` credentials list below.

5.  Change the IP address of `eth0` (obtained in Prep step 1) in the
    `/etc/network/interfaces` file and save:

        iface eth0 inet static
        address 10.180.21.79          <--CHANGE_ME
        netmask 255.255.255.0         <--CHANGE_ME
        network 10.180.21.0           <--CHANGE_ME
        broadcast 10.180.21.255       <--CHANGE_ME
        gateway 10.180.21.254         <--CHANGE_ME
        #dns-* options are implemented by the resolvconf package, if installed
        dns-nameservers 10.180.21.21 10.180.21.22       <--CHANGE_ME
        dns-search lab.lab

6.  Reboot the VM

7.  You can connect as root via ssh or the console into the Builder VM.

8.  When editing this file, search for the string **CHANGE_ME** and change them
to your appropriate information.  Edit the following lines in the
`/Builder/Packer/Contrail-AIO.json` file and save:

      1.  `line 32: remote_host` (ESXi server)
      2.  `line 33: remote_datestore` (on the ESXi server)
      3.  `line 35: remote_username` (ESXi user)
      4.  `line 36: remote_password` (ESXi password)
      5.  `line 40: ethernet0.networkName` (vSwitch name on ESXi)

9.    Start the build process by running:

      `cd /Builder/Packer`

      `packer build Contrail-AIO.json`

10.  This should start and complete the automatic Packer build process.  Depending
      on the resources of the ESXi server, it should take 25-45 minutes from
      start to finish.  You will see the output from Packer to builder cli while the
      builder scripts run.  At a high level it does the following:
      1.  Contrail-Builder runs in packer to create the VM itself on the ESXi server
      2.  Downloads the Ubuntu server ISO and calls a pre-seed config file to
      build and configure the OS.
      3.  Runs Ansible plays from a playbook to setup OpenStack and Contrail
      4.  Upon successful completion the VM will gracefully shutdown.

11.  Power up and log into the Contrail-AIO VM console or ssh.
12.  As `root` user, run the command `contrail-status`.  This will give you insight
     on all the services associated with contrail, all of which should show active.

     **NOTE** All services typically take ~5 minutes to start/restart.
     Additionally, if you only see 3-4 services listed, it means the install
     didn't fully complete.  You can manually re-run the installer from
     inside the VM without worry;

     * `cd /opt/contrail/utils` then run `fab setup_all`.
     * If `contrail-status` returns a `command not found`, or something similar
     you will need to run;
     * `cd /opt/contrail/utils` and the `fab install_contrail`.  Once that is complete,
     run `fab setup_all`

13.  You should now be able to access the Contrail and OpenStack GUIs:

GUI access
* OpenStack = http://IP_address_YOU_provided_in_Step_5/horizon
* Example http://10.10.10.10./horizon

* Contrail = https://IP_address_YOU_provided_in_Step_5:8143
* Example https://10.10.10.10:8143

User Crendentials - CLI  username/password
* contrail/contrail123
* root/contrail123

User Credentials - OpenStack and Contrail GUI
* admin/contrail123
