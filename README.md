[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/Raul-Flores/L3vpn-Multivendor)


#  Automatic L3vpn Multivendor with Ansible and NAPALM (IOS,XR,Junos,etc)

Automatically create and deploy an L3VPN service (template creation, configuration and validation that the network is in the way its configuration was planned and that there are no floating configurations), the l3vpn service model is loaded into an ansible playbook and based on The same model creates a template of configurations for each type of equipment involved in the service and then through NAPALM-Ansible library deploys the service to multiple network devices, in the example shown in the demonstration video on YouTube (I leave link below) an automated deployment of l3vpn to devices with IOS, XR, JunOS is shown and it is validated that they announce prefixes, BGP is used as PE-CE protocol, but any type of IGP can be used in the same way as PE-CE.

#  Technology stack

Python 3.x, Ansible 2.7 or higher, it can be run on windows as long as ubuntu as WSL is installed and from there run it

#  Status

Only one version already tested and validated in emulated enviroment (EVE-NG with multivendor ISP Topology, see the video link below)

# Use Case Description

Service providers that wish to implement l3vpn services in an automated way, with support for multivendor, generates a standard model for the deployment of the service, in the same way this tool serves to the companies that have this service and require to automate the configuration to many devices saving time in deployment and validation.

# Installation

Ubuntu 18.04 o any Distribution of Linux with support to Python3 and Ansible2.7 or higher (support with Ansible 2.9)
it can be run on windows as long as ubuntu as WSL is installed and from there run it (recomended)

# Steps to install in Ubuntu workstation (automation station)
<pre>
sudo apt-get install python3
sudo apt-get install ansible
sudo apt-get install git
sudo apt-get install python3-pip
sudo pip3 install virtualenv 
mkdir l3vpn-automation
python3.6 -m venv venv-l3vpn
source venv-l3vpn/bin/activate
git clone https://github.com/Raul-Flores/L3vpn-Multivendor.git
pip install requirements.txt
</pre>

# Configuration
you have to go to the main directory, some PEs devices are found as an example you can substitute their names and associate them in the file etc/hosts in your linux the name with the IP address you need.


# example
<pre>
 cat /etc/hosts
This file is automatically generated by WSL based on the Windows hosts file:
(output ommited for brevety)
The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
192.168.87.120 PE1    #(Change this in your hosts file for your devices)
192.168.87.47 PE3
192.168.87.24 PE4
192.168.87.129 PE2
(venv-l3vpn) rflores@Raul-Flores-Trabajo:~/venv-l3vpn/L3vpn-Multivendor/l3-vpn-svc
</pre>
# Structure
 <pre>
(venv-l3vpn) rflores@Raul-Flores-Trabajo:~/venv-l3vpn/L3vpn-Multivendor/l3-vpn-svc
$ tree
 ansible.cfg
 candidate_config
    PE1-config.txt
    PE2-config.txt
    PE3-config.txt
    PE4-config.txt
 deploy-service.yml
 diff
    PE1-diff.txt
    PE2-diff.txt
    PE3-diff.txt
    PE4-diff.txt
 generate-model
    tasks
       main.yml
    templates
       data-to-device-model.j2
    vars
        main.yaml
 generate-model.yml
 generate-template
    tasks
       main.yml
    templates
       ios
          core.j2
       iosxr
          core.j2
       junos
           core.j2
    vars
        main.yml
 generate-template.yml
 hosts
 requirements.txt
 </pre>
 
There are .txt files (you can delete them), the program automatically creates them so delete these files (in the folders of candidate_config and diff)

rm candidate_config/*.txt

rm diff/*.txt

delete the file name main.yml that is in the path: (generate-template/vars/main.yml) This file is generated so that the template generation model can build the equipment configurations, delete it as it will be created automatically from the service model data.
rm generate-model/vars/main.yaml

# **Usage**

to deploy the service once you have defined the equipment in the hosts file (here the user and password that will be applied by device type are defined), the names of the network devices correctly (in the file etc/hosts) and the devices have the configuration of SSH, then I would place the example commands to activate ssh and xml agent in XR, (this is important since if it cannot be reached through SSH it will not support it, the other examples of configurations by equipment (IOS, Junos) is beyond the scope of this information, (it is assumed that the engineer can configure this as it is something basic)

## configuration example in XR device
 <pre>
!
crypto key generate dsa
!
xml agent tty
iteration off
!
ssh server v2
!
 </pre>
(username and password admin by default)

once we have everything ready we will proceed to edit the variables in the file that is in the path: l3-vpn-svc/generate-template/vars/ main.yaml  (create it and name it as main.yaml if it is not found), change the data of the variables as you need your L3VPN service:

example of configuration (you need change the values, important to follow the same structure, in this example the minimum required to build an l3vpn service is assumed (does not include the configuration of the session towards the RR, You have currenct active session   MP-BGP in the up state with the PE to the RR):
<pre>
---
common:
  bgp_asn: 64512

vpn_service:
 - vpn_name: VPN-A
   route_distinguisher: '64512:65512'
   route_target: '64512:65512'
   neighbor_remote_as: '65512'

nodes:
  - name: PE1
    rid: 11.11.11.11
    vpn_int_mask: /30                 ====> put / format if the device is Juniper
    vpn_int_ip: 11.0.0.1              ====> put the ip address of the local interface (PE interface connect to the Client)
    vpn_bpg_neighbor_ip: 11.0.0.2     ====>put the ip address to the customer (its assume do you know what its this ip address)
    interface: em2              ==> put the name of interface that you use to connect to the client (what interface connect to the CE)

  - name: PE2
    rid: 22.22.22.22
    vpn_int_mask: 255.255.255.252
    vpn_int_ip: 11.0.0.5
    vpn_bpg_neighbor_ip: 11.0.0.6
    interface: GigabitEthernet1

  - name: PE3
    rid: 33.33.33.33
    vpn_int_mask: /30
    vpn_int_ip: 11.0.0.9
    vpn_bpg_neighbor_ip: 11.0.0.10
    interface: GigabitEthernet0/0/0/1

  - name: PE4
    rid: 44.44.44.44
    vpn_int_mask: /30
    vpn_int_ip: 11.0.0.13
    vpn_bpg_neighbor_ip: 11.0.0.14
    interface: GigabitEthernet0/0/0/1
</pre>
run the foloweeds commands:
<pre>
 ansible-playbook -i hosts generate-model.yml
</pre>
(this command create a main.yml file in the path: l3-vpn-svc/generate-template/vars/) please check if its correctly created.
<pre>
ansible-playbook -i hosts generate-template.yml
</pre>
(this command create all txt configs in txt in candidate_config file)
<pre>
ansible-playbook -i hosts deploy-service.yml
</pre>
(this command put the configuration using Ansible-NAPALM to the devices.)

# How to test the software
you can check the configuration in the devices, or in the RR (see the video example)

# Getting help

If you have questions, concerns, bug reports, etc., please create an issue against this repository, or send me a email
to: raul.flores@ciberc.com

# Link Video Example 

https://www.youtube.com/watch?v=MnZQZNcFetk&t=692s

![](https://github.com/Raul-Flores/L3vpn-Multivendor/blob/master/L3VPN.PNG)

