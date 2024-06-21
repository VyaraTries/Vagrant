Open your terminal (e.g. a Mac Terminal or Windows Command Prompt). Create a new directory and cd into it:
                       
                        mkdir myproject
                        cd myproject

Now use the vagrant init command to initialise a new vagrant machine.
We give the name of the “box” we want to use (a Vagrant box is basically a virtual machine template, with some software pre-installed):

                  vagrant init ubuntu/jammy64

This initialises a new Vagrant virtual machine, using Ubuntu version 22.04 (which has a codename of “Jammy Jellyfish”, hence the “jammy” bit).
       Then, boot up the virtual machine with:
                        
                        vagrant up


Vagrant works a little bit magically. How does it know which virtual machine you want to boot up?
When you run vagrant, it looks for a config file in your current directory, called a ‘Vagrantfile’. This is a file that describes a virtual machine (and we’ll see more about it later).

Let’s go inside the VM with this command:

                        vagrant ssh


Vagrant helpfully shares our project folder (the one that you created above), so that we can access it inside the virtual machine. It’s mounted at the path /vagrant:
This should show you one file, Vagrantfile. This file was created by Vagrant when we started our VM with vagrant init.

                        ls /vagrant

####################                        
Install Apache
####################
Remaining inside the VM, let’s list all the packages that are installed:


                  apt list --installed

Let’s install Apache HTTP Server. First search for it. In Ubuntu, we use the apt search command for this:


                        apt search apache

Before we install Apache, we should do a package update. This is good practice! It just ensures that we’ve got the latest updates for everything installed on our system:

                       SUDO apt update

Now let’s install Apache:

                  sudo apt install apache2

                  
                        
Once it’s finished installing, we can see which files were installed using the dpkg command:
This shows all the files installed onto the system by the ‘apache2’ package.

                              dpkg -L apache2


So apt has created and started a service for Apache.
Services are programs which are started and managed automatically by the operating system, so you don’t need to run them yourself.

Check if Apache is running and test
We can see the status of the Apache HTTP Server “service” using the systemctl command:

                        systemctl status apache2

Now let’s make a test request using the curl program. We’ll connect to localhost and port 80 (which is the default port for HTTP):

                        curl -v localhost:80

####################
Set up networking for our VM
To access our web server from outside the VM, we’ll need to give the VM an IP address that we can reach from outside the VM
####################

Inside the SSH session, type:

                              ip addr


The lines labelled with “inet” are actually the IP addresses. In this example, the machine currently has 2 x IP addresses:

127.0.0.1, which is an IP address that machines use to refer to themselves. You use this IP address when you want to access something on the machine, from the machine itself! Also called “localhost”.

10.0.2.15 (ignore the bit after the / for now). This is the machine’s IP address that it uses to identify itself to the network. We can use this to access it from our host computer.

#######
I tried the IP address but it’s ‘NAT’ working…
By default, VirtualBox configures your virtual machine with a network configuration called Network Address Translation (NAT).

NAT is a way to connect computers to a network, so that a machine can “see outside” (to the network & the internet), but to other machines, it appears invisible and unreachable.

(This is like how your laptop connects to your internet router at home.)

So, we need to change the network configuration, so that the machine is reachable.

Using DHCP instead
#######


####################
Configuring the VM’s network connection
Set the virtual machine’s IP address with DHCP
####################

Exit the virtual machine so you’re back at your terminal (or Windows command prompt).
Find the file myproject/Vagrantfile, and add this line to set the network configuration to DHCP:

                  config.vm.network "private_network", type: "dhcp"

(It goes between the Vagrant... and end blocks).

Now run this command to reload the configuration and restart the VM:

            vagrant reload

When the machine restarts, SSH into it again:
Now if you re-run the IP addr command, a new network interface has appeared!

This is like another virtual network connection, and it contains our VM’s IP address:

###############
Test the network setup
###############

Access the website from a web browser
Open a browser and go to http://IP_ADDRESS (e.g. for my example it would be http://172.28.128.3):
You’re accessing a web server, running inside a Linux virtual machine, on your laptop.


#############
Add some website content
#############
Edit the default home page

Use a text editor to edit the website home page
Let’s have a look in /var/www/html:
                        
                        cd /var/www/html
                        ls -al

Notice that there’s a file inside, index.html.
Now edit this file using an editor. nano is probably the easiest one to use, or if you’re feeling brave or knowledgeable you can use vi!
Now open the file with nano:

                        sudo nano index.html

Now change a part of the HTML.
Add a few words somewhere, edit the header, swap one of the images… it’s up to you. Make a change where you’ll notice it – perhaps in the title tag or the main header.

###############
Replace the default website
###############
Now we want to replace the default web site content with our actual website.
We can copy some files into the virtual machine with Vagrant.
With Vagrant, whenever you start a virtual machine with vagrant up it automatically syncs the contents of the current directory into a directory called /vagrant on the virtual machine.

Copy across the new website
Create a directory called webcontent underneath the location where you ran vagrant init.

                        mkdir webcontent

Inside the virtual machine, go to the /vagrant directory and type ls to list all the files:

                  cd /vagrant
                  ls

Let’s copy the website files to /var/www/html.
Again, we’ll need to use sudo, because the target directory is owned by root. So inside the VM, run this:
This will copy the contents of webcontent into the html directory.


                  sudo cp /vagrant/webcontent/* /var/www/html/

Now reload your web browser again. The web server now shows our web site, instead of the default page!

###############
Automate our work
###############
Instead of relying on manual work, let’s automate everything. Whenever the virtual machine is created, we want to automatically install Apache HTTP Server and copy the website files.
This is done with a process called provisioning.

Add provisioning commands
In Vagrant, a block of provisioning commands begins with config.vm.provision.
Add some provisioning steps to the virtual machine

In the Vagrantfile, add the provisioning code like this, just before the end command in the Vagrantfile:

                        config.vm.provision "shell", inline: <<-END
                        apt update
                        apt install -y apache2
                        cp -r /vagrant/webcontent/* /var/www/html/
                        echo "Machine provisioned at $(date)! Welcome!"
                        END


Let’s check that the Vagrantfile is looking good.
                        
                        vagrant validate

Run vagrant destroy to delete the existing virtual machine. 💣

                        vagrant destroy

Now finally bring up the machine with the automatic provisioning code:

                        vagrant up


You’ll get a lot of log output, and the final line should read something like…

                        default: Machine provisioned at Sun May 15 14:43:22 UTC 2022! Welcome!

You’ve done it! You have created a Linux virtual machine, with Vagrant, that automatically provisions a web server and a website.









1. Install Apache in the Ubuntu VM
      List installed packages.
      Search for the Apache package.
      Update the package list with root privileges.
      Install Apache.
      Check installed files for Apache.
2. Verify Apache Installation
   Check the status of the Apache service.
    Test Apache with a curl request to localhost.
3. Set Up Networking for VM
    Check the current IP address.
    Configure the VM's network connection to use DHCP:
    Exit the VM.
    Add config.vm.network "private_network", type: "dhcp" to the Vagrantfile.
    Reload the VM configuration with vagrant reload.
    SSH back into the VM and re-check the IP address.
4. Test Network Setup
    Access the website from a web browser using the new IP address.
5. Edit Default Website Content
    Navigate to /var/www/html and list files.
    Edit index.html with a text editor.
    Save changes and reload the web page to see updates.
6. Replace Default Website
    Create a webcontent directory.
    Create or download website content.
    Copy the website files to /var/www/html inside the VM.
7. Automate the Setup
    Add provisioning steps to the Vagrantfile:
    Install Apache.
    Copy website files to the correct location.
    Print a provisioning confirmation message.
    Validate the Vagrantfile.
    Destroy the existing VM.
    Bring up the VM with the new provisioning code to automatically install Apache and set up the website.


With these steps, you've set up and automated the provisioning of a Linux VM with Apache HTTP Server, ensuring your website is served as soon as the VM is started.








