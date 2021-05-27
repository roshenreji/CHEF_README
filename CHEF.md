#### CHEF DOCS: https://docs.chef.io/

#### Chef setup | Chef installation on AWS | Chef Tutorial for beginners Links :
https://www.youtube.com/watch?v=JwjBZkxjaxE&list=LL&index=4


# Setting Up Chef Server
* Open an account on CHEF Manage
* This will be your hosted server for CHEF
* Site Link: https://manage.opscode.com

# Creating a WorkStation
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

# Connecting workstation with CHEF Server
* Head over to CHEF Manage Server
* Select the Adminstration Tab
* On Actions select 'Starter Kit'
* Download the starter kit on local system
* UnZip that File
* Then copy the unzipped File into the workstation
* Make sure you have the .pem file in the same folder from where u want to copy the file
* open cmd in the pwd
```
scp -r -i "awskeypair.pem" chef-starter/chef-repo/ ubuntu@ec2-3-12-36-226.us-east-2.compute.amazonaws.com:~/
```
* Copy the .pem file to that folder and make sure the path to be pasted in is correct
```
scp -r -i "awskeypair.pem" awskeypair.pem ubuntu@ec2-3-12-36-226.us-east-2.compute.amazonaws.com:~/chef-repo
```
* These all steps will help to communicate workstation and server

* Use this command for verifying ssl configuration for the CHEF Infra Server (Run this from the chef-repo directory)
```
knife ssl check
```


# Bootstrap a node (From workstation)
* Create two node vms on aws/azure and tag them as Node1 and node2
* Remember to add the inbound access for HTTP 80 in the sec groups of both nodes
```
knife bootstrap <public ip address of node> --ssh-user ubuntu --sudo --ssh-identity-file awskeypair.pem -N <node name>
```
* Head over to CHEF Server 
* Click on Nodes Tab
* You will be able to see the node list
* Commands for node details
```
knife node list

knife node show <Node Name>
```
* Link: https://docs.chef.io/install_bootstrap/ (Not So Imp)




# Creating a Sample CookBook (From Workstation)

* Either create a cookbook inside 'cookbooks folder' or create a new directory and create cookbooks inside them
* Run the below command to create cookbook inside chef-repo/cookbooks
```
chef generate cookbook <cookbook name>
```

* Sample Scripts inside recipe directory :

(You can create your own .rb recipe file or else use default.rb file)
```
chef generate recipe <recipe_name>
```
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


# Uploading a CookBook (From WorkStation)
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
* Add the cookbook to the Node's Run List (adds on top of previous run_lists)
```
knife node run_list add <node_name> recipe[<cookbook_name>::<recipe>]

Ex:
knife node run_list add Node1 recipe[simplecookbook::default]

```
* Once added then all future pushes of cookbooks will automatically update in the run list of the node.
* Note: You dont have to specifically run this command and again

* Set the cookbook to the Node's Run List (removes previous recipes from run_list)
```
knife node run_list set <node_name> recipe[<cookbook_name>::<recipe>]
```
* Removing recipes from run_list
```
knife node run_list set <node_name> ''
```

#### Downloading and Uploading cookbooks from supermarket

```
knife supermarket downlaod <cookbook_name>

tar -xzf <file>.tar.gz

knife cookbook upload <cookbook_name>
```

# Creating Attributes

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


# Node Attributes

* Create a default attribute inside a cookbook
```
chef generate attribute default

vim attributes/default.rb

default['myname'] = 'atib'
``` 
* Add file resource in recipe
```
file "home/ubuntu/nodeattribute.txt" do
    content "My Name is #{node['myname']}" 
end
```
* Node attributes can be set directly from workstation by using command

```
knife node edit <nodeName> --editor /usr/bin/vim  (double TAB to check availability)

or

knife node edit <nodeName> --editor vim

```
* This will open a json file to edit :
```
{
  "name": "Node1",
  "chef_environment": "development",
  "normal": {
    "myname": "Vikas",
    "tags": [

    ]
  },
  "policy_name": null,
  "policy_group": null,
  "run_list": [
  "recipe[linux_node::userinfo]"
]

}

```
* Can check node attributes by using command
```
knife node show <nodeName> -F json
```
* Then run chef-client on that node


# Working with Environments

Refer Link: https://docs.chef.io/environments/

* Checking list of environments
```
knife environment list
```

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
knife environment from file dev.json
```

* For Production (prod.json)
```
{
   "name": "production",
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
                "env": "production",
                "name": "Atib"
           }
   }
}
```
* Run This Command to update Environemnet from environments directory:  
```
knife environment from file prod.json
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

# Roles

* List the Roles present in chef server
```
knife role list
```
* Details of Role
```
knife role show <role_name>
```
* Creating a new Role
```
export EDITOR=$(which vi)

knife role create <role_name>
```
* Editing Roles
```
knife role edit <role_name>
```
* Editing a Role (demo.json)
```
{
  "name": "demo",
  "description": "A simple demonstration role.",
  "json_class": "Chef::Role",
  "default_attributes": {

  },
  "override_attributes": {

  },
  "chef_type": "role",
  "run_list": [
    "recipe[pluralcookbook::default]"
  ],
  "env_run_lists": {

  }
}
```
* Here a run_list is assigned to a role.
* You can add many recipes in the run_list of a single role and assign the role to a node.

* Assigning Role to a Node 
```
knife node run_list set <node_name> 'role[<role_name>]'
```
* Deleteing a Role
```
knife role delete <role_name>

Upon Confirmation? Type Y
```

### Another way to create Roles

* cd into roles dir
* create a role file : <role_name>.json
* Run Command from roles dir
```
knife role from file <role_name>.json
```


#### Precedence
*************
(for default)
attributes <- node object <- environment <- role

(for normal)

(for override)    
*************

# DataBags

A directory named data_bags is created.
For each data bag, a sub-directory is created that has the same name as the data bag.
For each data bag item, a JSON file is created and placed in the appropriate sub-directory.


* Folder Structure
```
Repo      > data_bag folder > databags > items

chef-repo > data_bags       > example
                            > users    > kate.json
                                       > james.json
```

* Listing databags
```
knife data bag list
```
* Creating databags
```
export EDITOR="vim" or "nano"

knife data bag create <databag_name> <databag_item_name>
```
* Showing Items in databags
```
knife data bag show <databag_name>

knife data bag show <databag_name> <databag_item_name>
```
* Editing items in databags
```
export EDITOR="vim" or "nano"

knife data bag edit <databag_name> <databag_item_name>
```
* Sample databag item example 

```
knife data bag create users james


{
  "id": "james",
  "fullName": "James Bannan",
  "firstName": "James",
  "lastName": "Bannan",
  "country": "Australia",
  "city": "Melbourne"
}

Save
```

* Deleting databags items
```
knife data bag delete <databag_name> <databag_item_name> 
```

* Recipe calling databag items (refer linux_node/recipes/userinfo.rb)
```
users = data_bag('users')

users.each do |user|
  userobject = data_bag_item('users', user)

  file "home/ubuntu/#{userobject['id']}.txt" do
    content "The user is #{userobject['fullName']}, they live in #{userobject['city']} which is in #{userobject['country']}."
    mode '00755'
  end
end

```
* Run chef-client on nodes

### Another way to create databags 

* cd into data_bags dir
* create a data_bag
* create a data_bag_item : <filename>.json
* Run Command from data_bags dir
```
knife data bag from file <data_bag_name> <data_bag_name>/<filename>.json
```


# Encrypting a data bag item with shared keys

#### SECRET KEY GEN
* Encrypting a data bag item requires a secret key.
* OpenSSL can be used to generate a random number, which can then be used as the secret key:
* Note: This command here is run inside chef-repo and hence the same path will be used later.

```
openssl rand -base64 512 | tr -d '\r\n' > my_secret_key

```
#### ENCRYPT

* If new item is to be created and encrypted
```
knife data bag create <data_bag_name> <data_bag_item_name> --secret-file ~/chef-repo/my_secret_key

```
* If item is already present and is to be encrypted
```
knife data bag edit users emily --secret-file ~/chef-repo/my_secret_key

```
#### VERIFY
```
knife data bag show users emily
```
#### DECRYPT
```
knife data bag show --secret-file ~/chef-repo/my_secret_key users emily
```


# CHEF Vaults
```
knife vault list

knife vault create user james

knife vault create user kate -J kate.json
```





# Custom Resources
* Dont have to create resources folder explicitly if already not present
```
chef generate resource ~/chef-repo/cookbooks/<cookbook_name> <resource_name>

```
#### EXAMPLE01
* Assume the cookbbok name is : pluralcookbook
* vim into the resource_file (site.rb)
```
property :homepage, String, default: '<h1>Say Hello To Custom Resources</h1>'

action :create do
        package 'apache2'

        service 'apache2' do
                action [:enable, :start]
        end

        file '/var/www/html/index.html' do
                content new_resource.homepage
        end
end
```
* Edit Recipe File to use custom resources
* Reference : <cookbook_name>_<custom_resource_name>
```
pluralcookbook_site 'apache2' do
        homepage "<h1>This Site is running on #{node['hostname']} and the Operating System is #{node['platform'].capitalize()}</h1>"
end
```
* Upload cookbook and run chef-client from node


#### EXAMPLE02
* Assume the cookbbok name is : node_site
* vim into the resource_file (website.rb)

```
property :homepage, String, default: '<h1>Hello world!</h1>'

action :create do
  folder = '/var/www/html/'
  file = 'index.html'


  if platform_family?("debian")
    apt_update
  end

  package 'Install Apache' do
    case node[:platform]
    when 'redhat', 'centos'
      package_name 'httpd'
      action :upgrade
    when 'ubuntu', 'debian'
      package_name 'apache2'
      action :upgrade
    end
  end

  service 'Start Apache' do
    case node[:platform]
    when 'redhat', 'centos'
      service_name 'httpd'
      action [:enable, :start]
    when 'ubuntu', 'debian'
      service_name 'apache2'
      action [:enable, :start]
    end
  end

  file "#{folder}#{file}" do
    content new_resource.homepage
  end
end

action :delete do
  package 'httpd' do
    action :delete
  end
end
```
* Note that here default action is create
* Delete action also can be explicitly called

* Edit Recipe File to use custom resources
* Reference : <cookbook_name>_<custom_resource_name>


* default.rb
```
include_recipe 'node_site::site'
```
* site.rb
```
node_site_website 'httpd' do
    homepage "<h1>This site is running on #{node['hostname']} and the operating system is #{node['platform'].capitalize()}.</h1>"
end

```
* Upload cookbook and run chef-client from node

# Kitchen Commands
```
kitchen list

kitchen create

kitchen converge

kitchen login (logging onto vm)

kitchen verify (testing)

kitchen destroy

kitchen test (combines all processes viz. create+converge+verify+destroy)
```


#### Chef Automation Tutorials-6 | Configuration Management Links :
https://www.youtube.com/watch?v=doS1p5AR6KI&list=LL&index=2&t=1886s
