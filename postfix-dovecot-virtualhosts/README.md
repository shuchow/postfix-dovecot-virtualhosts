# Introduction
This playbook will set up email accounts for multiple domains on one IMAP server.  It installs and configures dovecot and postfix for virtualhosts.

# Features:
1. Domains and email address are stored in MariaDB/MySQL for easy maintenance.
2. Encryption set up through Let's Encrypt.
3. Credentials stored in Ansible Vault, allowing repo to be source controlled.

# Things You Should Know

1. This does not set up SMTP services to send email.  I use Amazon SES as an SMTP server, but there are many others out there.  There are many reasons I don't run my own SMTP server, but the main one is that DKIM for virtual domains is hard.  

2. This method requires MariaDB, MySQL, or Postgres.  Most commonly used is MariaDB/MySQL.  You should know how to create a database and grant a user permissions on it.  This playbook does not install or setup the database since it is so common to have one of these installed.  However, a role to install and setup MariaDB is included if needed.

3. You should know how to change your domains' DNS records. 

4. The Certbot playbook uses DNS Challenge.  That means it automatically uses your registrar's API to update your domain's DNS records and verify before generating the cert.  This is done by official and community plugins.  This repo currently supports Gandi and Route53.  PRs for others are welcome. 

5. Basic Linux administration including Ansible.
 
# Things You Need to Do
1. Set up an Ansible Vault with secrets.  See "Setting up the Vault" below.

2. Modify host_vars/hostserver.com.yml.  See "Setting up Your Hostvars."  This is where your domains are stored.

3. Set up a database and give a user permissions on it.  An SQL file is included in this repo to set up the database, but remember to set up a user/password and give the user SELECT privileges on it.

4. Add the following DNS records to each domain:
    
    a. root domain ("@") MX record pointing to "10 mail.yourdomain.com" (assuming you're using the mail. convention).
    
    b. "A" name record of "mail" or "mail.yourdomain.com" depending on your registrar, pointing to your server's IP address.


## Setting up the Vault
1. Edit the global-vault.yml file with the required variables.  If something is not used, like the gandi_token, leave the value blank, but leave the key.

2. Encrypt the file, leaving it in the same directory.
> ansible-vault encrypt global-vault.yml

## Setting up Your Hostvars

# Adding Users and Domains

# Running the Playbook
Run from this directory.

> ansible-playbook --key-file ~/.ssh/{{ your private key }} --vault-id prod@global-vault.yml --extra-vars "add_user=true" --ask-vault-pass playbooks/setup_mail.yml 
