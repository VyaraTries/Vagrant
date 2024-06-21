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
