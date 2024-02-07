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
 
# Steps You Need to Do
1. This repo stores all sensitive data in a centralized encrypted vault.  See "Setting up the Vault" below.

2. Modify hosts and host_vars/hostserver.com.yml.  See "Setting up Your Hosts."  This is where your domains are stored.

3. Set up a database and give a user permissions on it.  An SQL file, database.sql, is included in this repo to set up the tables.  This file is a from a MySQL dump, so modifications may need to be made for PostGres.  After the database and tables are set up, create a user/password and give SELECT privileges on it.

4. Add Domains.  See "Adding Domains" below.

5. Run the playbook.  See "Running the Playbook."

6. Add Users.  See "Adding Users" below.

7. Connect with an email client.

## Setting up the Vault
1. Edit the global-vault.yml file with the required variables.  If something is not used, like the gandi_token, leave the value blank, but leave the key.

> &ndash; &ndash; &ndash;
> 
> gandi_token: //Gandi API Token.
>
> db_user: //Database user
>
> db_password: //Database password
>
> db_name: //Name of the database.
>
> db_host: //Host server of the database.

2. Encrypt the file, leaving it in the same directory.
> ansible-vault encrypt global-vault.yml

## Setting up Your Hosts

Hosts are added to the hosts file, but it has repercussions requiring editing other files.

1. hosts: Standard ansible hosts file.  The name in the bracket ([myserver]) are group names in ansible.  You can name this whatever you want.  

2. playbooks/setup_mail.yml: The group name form #1 is referred to in the playbook, so if you change the group name, you need to change it in the "hosts:" key of playbooks/setup_mail.yml.

3. Underneath the group name is the server name or IP address of the server.  This allows a playbook to be run on multiple servers, but the method in this repo only works for one server, so only one line should be under here.

4. host_vars/{{ server_name_or_ip.yml }}: The file under host_vars directory should be whatever the server name IP address specified under the group name in #3, followed by ".yml". 

## Adding Domains

1. For each domain, add the following DNS records:
    
    a. root domain ("@") MX record pointing to "10 mail.yourdomain.com" (assuming you're using the mail. convention).
    
    b. "A" name record of "mail" or "mail.yourdomain.com" depending on your registrar, pointing to your server's IP address.

2. Set up the wildcard_domains field in host_vars/{{ server_name_or_ip.yml }}.  This field specifies the domains and settings to be used in the roles.  Each domain is a object in this field.  

      a. domain: The domain.  No "www." or "mail."

      b. enabled: Set to true.  Saved for future use.

      c. registrar: Currently supports Gandi "gandi" and Amazon Route 53 "route53"

> wildcard_domains:
> 
> &ndash; { domain: "exampleone.com", enabled: true, registrar: "gandi" }
> 
> &ndash; { domain: "exampletwo.com", enabled: true, registrar: "gandi" }
> 
> &ndash; { domain: "examplethree.com", enabled: true, registrar: "route53" }

3. Add each domains to the database's "domains" table.


## Running the Playbook
Run from this directory.

> ansible-playbook --key-file ~/.ssh/{{ your private key }} --vault-id prod@global-vault.yml --extra-vars "add_user=true" --ask-vault-pass playbooks/setup_mail.yml 

## Adding Users and Domains

Every user needs to be added to the database.

1. Use doveadm to generate a password.  You may need to sudo for this.

> doveadm pw -s SHA512-CRYPT

You will be prompted for the password and asked to confirm.  You'll get an output like this:

> {SHA512-CRYPT}$6$8QtkhwU4C1WQqFtf$SYVGGr.nhj123nHmIPNk1Zsr086rwzCyi.yWTQn123UnjfUjA1mhDETR6.pYryzDm2d.EkOyVm123kDUPY1gy0

Save everything after the "{SHA512-CRYPT}" prefix.  That is the hashed password.

2. In the database, add the email address, the hashed password from above, and quota if overriding the default, to the "users" table.

## Connecting to the server.

You should now be able to connect through IMAP to the server with an email client.  This should be done over port 143.  Select TLS/SSL in your client.

# Registrar Notes

## Gandi

You will need a Personal Access Token. Put this token in the "gandi_token" field in the global-vault.yml.

1. Login in to the Gandi Dashboard.  

2. Click on Organizations.

3. In Personal Access Tokens, Select "Create a token."

4. Fill in the fields, set expiration to 1 year.  Give the token permissions to "Manage domain name technical configurations" and "See and renew domain names."

5. Create the token and save it to the global_vault.

## Route 53

You will need to create an identity, create a policy that allows modification to the domain, give the user access to the policy, get a secret key and token, and configure it in the root user's AWS configuration on the server.

1. Login in to the Amazon Dashboard, go to Identity and Access Management (IAM).

2. Create a policy that allows you to modify domains.

    a. Click on "Policies" in the left nav bar.

    b. Select "Create a Policy."

    c. "Select a Service": find and pick "Route 53."

    d. In "Actions Allowed," find and select the following:
        - ChangeResourceRecordSets (Write)
        - GetChange (List)
        - ListHostedZones (List)

    d. Name the policy and save it.  Go back to IAM.

3. Create a user and give it the new policy.

    a. Click on "Users."

    b. Name the user in step 1.

    c. In step 2, "Set Permissions," select "Attach policies directly"

    d. Find the new policy and select it.

    e. Click "Next."

    f. Confirm and create the user.

4.  Create an access key and secret access key for that user.

    a. In the list of users, click on this new user.

    b. Click "Create Access Key."

    c. Select "Third Party Service" and click on the confirmation check box.

    d. Select "Create Access Key."  Copy the Access Key.  The Secret access key will be in "**************"  Click "Show" next to it.  Copy that too.

5. Set up the AWS credentials for the root user on the host.  You must be root to do these steps:

    a. In /root, create a directory named .aws.

    b. In .aws, create a file named config.  In it, populate the access key and secret access key from above.

   >[default]
   >
   > aws_access_key_id=THEACCESSKEY
   >
   > aws_secret_access_key=THESECRET

    c. Create it and give the file read/write access only to root.

    > chmod 500 config