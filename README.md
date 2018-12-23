# Ansible-Install-Wordpress

Ideally, we would want there to be a server set aside for running Ansible playbooks, we will call this our build-server. Here our Ansible playbooks would reside and then we would have target servers on which we would like to install WordPress.

Step 1: The Server(s) Setup
To allow Ansible to connect to our WordPress server, you would need to have a non-root user with sudo privileges. To allow this, create ssh keys on your build server.




