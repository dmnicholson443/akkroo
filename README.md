# akkroo

# PROJECT OVERVIEW

A simple application that will create three vagrant virtual machines for
testing and development purposes, all running the CentOS Linux distribution.  
Two are hosting a simple web application and one is a load balancer.  
All machines use NGINX for web hosting and load balancing.  Ansible is used to
provision and orchestrate the virtual machines.

## PREREQUISITES AND INSTRUCTIONS

Below are simple instructions to get the project up and running.  Firstly, this
section will detail the prerequisites that are required, and will detail the
versions used to build this project, but other versions may also work although
it should be noted other versions have not been tested in this project.

Other providers can be used, and these can be defined in the created vagrantfile,
in the machine.vm.provider section.

- Vagrant - 2.1.2
- VirtualBox - 5.2.16
- Ansible - 2.6.2
- NGINX - 1.12.2
- Linux - CentOS Linux release 7.5.1804 (Core)

## ENVIRONMENT

This project will create three vagrant virtual machines.  The environment is:

- machine1    172.16.100.101/24   web server
- machine2    172.16.100.102/24   web server
- machine3    172.16.100.103/24   load balancer

## INSTALLATION

This is a rather simple application, and so therefore requires only simple
instructions.  It is important that the local hosts file on the host is
updated with the correct IP address to hostname mappings.  I have created a
shell script that will match the three vagrant machines to IP addresses, although
should you wish to use different names and IP addresses, or increase the number
of vagrant machines, then the hosts file should be updates to reflect this
change.

Firstly, then, run the file "conf-etc-hosts.sh" as root.  Once completed,
you may verify the change or modify as necessary.

Next, if Oracle VirtualBox is not installed, please install this in accordance
with your operating system instructions.  The version above is recommended.

Following VirtualBox, ensure Ansible is installed on your host system.

Finally, install Vagrant, and again, following the instructions of your particular
operating system.

Create a folder of your choosing.  From within this folder, run the command:

```
git clone https://github.com/dmnicholson443/akkroo.git
```
Once the repository is cloned, you will have everything you need to run the
environment.

## RUNNING VAGRANT

The vagrantfile is already present and so it is not necessary to run
```
vagrant init
```
Note this environment has been written for CentOS 7.  Different CentOS versions
or Linux distributions may require changes to the vagrantfile, and the
Ansible playbook(s).  Please make these changes before proceeding.

Now, run the command
```
vagrant up
```
The first time this is run, a modified version of CentOS 7 is downloaded
from VagrantCloud and stored in the .vagrant folder.

This will create 3 virtual machines in VirtualBox, allocate their IPs, and
run the provision.yml playbook.

Upon completion, we can run
```
vagrant status
```
and we should see

*machine1	  running (virtualbox)
**machine2	running (virtualbox)
machine3  	running (virtualbox)*

If not, something might have gone wrong in the provisioning (not unheard of!).
In this case, it is better to start again by removing the environment,
```
vagrant destroy
```
clearing the cache (not always necessary but never a bad idea),
```
vagrant global-status --prune
```
and finally, re-installing the environment,
```
vagrant up
```

## TESTING LOAD BALANCING

The test is a simple one.

The load balancer is configured to listen on
http://www.dmninternal.com:8008
And forwards the request to either of the two web servers:
*172.16.100.101
172.16.100.102*
Each web server responds with a simple HTML file that details the name of
the server, **web01** and **web02** respectively.  By using a browser, and entering
the above URL and refreshing the page, we can prove the load balancer is working
when the request alternates between the two web servers, and therefore returning
the server name in the HTML markup response.  We can also use *curl*:
```
curl http://www.dmninternal.com:8008
```
where the response will be:
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Fri, 10 Aug 2018 09:33:09 GMT
Content-Type: text/html
Content-Length: 97
Connection: keep-alive
Last-Modified: Fri, 10 Aug 2018 08:21:41 GMT
ETag: "5b6d4b15-61"
Accept-Ranges: bytes

sh-3.2# curl http://www.dmninternal.com:8008
<html>
  <head>
    <title>WEB01</title>
  </head>
<body>
  <p>THIS IS WEB01</p>
</body>
</html>

**note WEB01 above**

and running the same command again:

curl http://www.dmninternal.com:8008
<html>
  <head>
    <title>WEB01</title>
  </head>
<body>
  <p>THIS IS WEB01</p>
</body>
</html>
sh-3.2# curl http://www.dmninternal.com:8008
<html>
  <head>
    <title>WEB02</title>
  </head>
<body>
  <p>THIS IS WEB02</p>
</body>
</html>

**now WEB02**

## TROUBLESHOOTING

This is a simple application, and so there is not much that can go wrong!

However, this project requires an Internet connection for the initial
configuration of vagrant, and running the provision.yml playbook.  If the
playbook fails, please ensure there is an Internet connection.

If it still continues to fail, turn on verbose output.  Modify the vagrantfile
in the *machine.vm.provision* section with:
```
ansible.verbose = true
```
Typically, it will fail if it cannot locate the files it needs to copy.
Examine the error message, and place the files it requires in the folders
it is searching.

Luckily, we are able to SSH in to Vagrant virtual machines for further troubleshooting.

Run the command:
```
vagrant ssh <vm>
```
substituting <vm> with the name of the virtual machine.

If NGINX fails to start, run the command:
```
sudo tail \var\log\nginx\error.log 20
```
and correct where necessary.
