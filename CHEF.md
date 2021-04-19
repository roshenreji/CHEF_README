#### CHEF DOCS: https://docs.chef.io/

#### Chef setup | Chef installation on AWS | Chef Tutorial for beginners Links :
https://www.youtube.com/watch?v=JwjBZkxjaxE&list=LL&index=4


## Setting Up Chef Server
* Open an account on CHEF Manage
* This will be your hosted server for CHEF
* Site Link: https://manage.opscode.com

## Creating a WorkStation
* Provision an ubuntu vm on aws/azure 
* Open Link: https://docs.chef.io/workstation/install_workstation/
* Head over to Linux Docs Part
* Copy those two commands for Debian/Ubuntu one by one and run on vm
* Run These Commands
```
sudo apt update

wget https://packages.chef.io/files/stable/chef-workstation/21.2.278/ubuntu/20.04/chef-workstation_21.2.278-1_amd64.deb

sudo dpkg -i chef-workstation_21.2.278-1_amd64.deb

```
* Verify Installation
```
chef -v
```

## Connecting workstation with CHEF Server
* Head over to CHEF Manage Server
* Select the Adminstration Tab
* On Actions select 'Starter Kit'
* Download the starter kit on local system
* UnZip that File
* Then copy the unzipped File into the workstation
* Make sure you have the .pem file in the same folder from where u want to copy a file
```
scp -r -i "awskeypair.pem" chef-repo/ ubuntu@ec2-3-12-36-226.us-east-2.compute.amazonaws.com:~/
```
* Copy the .pem file to that folder and make sure the path to be pasted in is correct
```
scp -r -i "awskeypair.pem" awskeypair.pem ubuntu@ec2-3-12-36-226.us-east-2.compute.amazonaws.com:~/chef-repo
```
* These all steps will help to communicate workstation and server

* Use this cmd for verifying ssl configuration for the CHEF Infra Server (Run this from the chef-repo directory)
```
knife ssl check
```


## Bootstrap a node (From workstation)
* Create two node vms on aws/azure and tag them as Node1 and node2
* Remember to add the inbound access for HTTP 80 in the sec groups of both nodes
```
knife bootstrap <public ip address of node> --ssh-user ubuntu --sudo --ssh-identity-file awskeypair.pem -N <node name>
```
* Head over to CHEF Server 
* Click on Nodes Tab
* You will be able to see the node list
* Link: https://docs.chef.io/install_bootstrap/ (Not So Imp)

#### Some ad-hoc commands
```
knife node list

knife node show <Node Name>
```


## Creating a Sample CookBook (From Workstation)
* Either create a cookbook inside 'cookbooks folder' or create a new directory and create cookbooks inside them
* Run the below command to create cookbook inside chef-repo/cookbooks
```
chef generate cookbook <cookbook name>
```
* Sample Scripts inside recipe directory :

(You can create your own .rb recipe file or else use default.rb file)
* Script1 to install Apache Server
```
package 'apache2'  do
  action :install
end
file '/var/www/html/index.html' do
  content '<html>This is a placeholder for the home page.</html>'
  action :create
end
service 'apache2' do
  action [:enable,:start]
end
```
* Script2 to install Nginx Server
```
package 'nginx' do
 action :install
end
service 'nginx' do
 action [:enable,:start]
end
```
#### NOTE: You can change versions of cookbooks in metadata.rb in your cookbook folder if required
#### Chef Cookbook Recipe Tutorial for beginners
https://www.youtube.com/watch?v=wY6xg7CI5Xw&list=PLsgnv1SN76IJIiBg0e1lAIIAW1xZXPHF1&index=4


## Uploading a CookBook (From WorkStation)
* For Syntax Check of ruby script
```
chef exec ruby -c recipes/default.rb
```
* Command to push CookBook (Always work from chef-repo folder)
* If cookbooks are present inside default folder 'cookbooks'
```
knife cookbook upload -a

OR

knife cookbook upload simplecookbook

```
* If cookbooks are present in user created folders then specifying path is mandatory for uploading
```
knife cookbook upload --cookbook-path ~/chef-repo/democook/ -a

OR

knife cookbook upload --cookbook-path ~/chef-repo/democook/ simplecookbook

```
* Add the cookbook to the Node's Run List
```
knife node run_list add Node1 recipe[<cookbook name>::<ruby file name inside recipe>]

Ex:
knife node run_list add Node1 recipe[simplecookbook::default]

```
* Once added then all future pushes of cookbooks will automatically update in the run list of the node.
* Note: You dont have to specifically run this command and again

## Creating Attributes
* Create a folder named attributes inside the cookbook you are working with.
NOTE: make sure recipes and attributes folder are on same directory structure
* create a default attribute file: default.rb
```
default['<cookbook_name>']['<variable_name>'] = '<value>'
```
EX:
```
default['samplecookbook']['env']= 'default'
default['samplecookbook']['name']= 'nobody'
```
* Add these attributes on recipe's default.rb file
```
var1 = node['<cookbook_name>']['<attribute_variable_name>']
var2 = node['<cookbook_name>']['<attribute_variable_name>']
```
Sample Recipe Example:
```
var1 = node['samplecookbook']['env']
var2 = node['samplecookbook']['name']
package 'apache2'  do
  action :install
end
file '/var/www/html/index.html' do
  content "<html>This is a placeholder for the #{var1} #{var2} page..!</html>"
  action :create
end
service 'apache2' do
  action [:enable,:start]
end

```
* Upload the cookbook after adding attributes

## Working with Environments
Refer Link: https://docs.chef.io/environments/
* Create an evironments directory in chef-repo directory
```
mkdir environments
cd environments
```
* Create a .json or .rb file inside the environments directory
* For Development (dev.json)
```
{
   "name": "development",
   "description": "",
   "cookbook_versions": {
           "<cookbook_name>": "= <cookbook_version>"
   },
   "json_class": "Chef::Environment",
   "chef_type": "environment",
   "default_attributes": {
   },
   "override_attributes": {
           "<cookbook_name>": {
                "env": "development",
                "name": "Isha"
           }
   }
}
```
* Run This Command to update Environemnet : 
```
knife environments from file dev.json
```

* For Production (prod.json)
```
{
   "name": "production",
   "description": "",
   "cookbook_versions": {
           "samplecookbook": "= <cookbook_version>"
   },
   "json_class": "Chef::Environment",
   "chef_type": "environment",
   "default_attributes": {
   },
   "override_attributes": {
           "samplecookbook": {
                "env": "production",
                "name": "Atib"
           }
   }
}
```
* Run This Command to update Environemnet from environments directory:  
```
knife environments from file prod.json
```

* Select the environment from the CHEF Manage Site manually for each node
* Click on Node > Details Tab > Environment Menu
* Select first node for Development and save it.
* Select second node for Production and save it.

* After Updating Environments run this from dev and prod nodes cli
```
sudo chef-client
```
* Open browser and put <ip addr for dev node>:80 and put <ip addr for prod node>:80 in two tabs
* check the results.




#### Some adhoc commands
```
knife environment list
```
#### Chef Automation Tutorials-6 | Configuration Management Links :
https://www.youtube.com/watch?v=doS1p5AR6KI&list=LL&index=2&t=1886s


