# AWS ECR LDAP #

# What is this ? 

Since, We are preparing to migrate all our services for authentication with OKTA (SS0) so Docker Registry system is one of them

In current state, We run a instance for docker registry where it authenticated with local password which We setup before. 

2 main disadvantages of this architecture are 
  - It does not scale to adapt with our growth 
  - The authentication simplicity which was not followed to SS0 mechanism being used 

So A(ws)-E(cr)-L(dap) is the solution to integrate AWS ECR with LDAP authentication for the users 

Basically, We will use AWS ECR service instead of Docker Registry self-hosted. But having bit modification for LDAP authentication with Nginx as frontend to allow people pull & push image. Developers just need to remember their own LDAP account to interact with AWS ECR system at behind. 

# How to setup ?

To clone repository 

```git clone github.com/toanc/aws-ecr-ldap```
 
To configure your environment with specified url - binddn & password following your own LDAP system
``` vi configs/nginx/nginx.conf 

 ldap_server ldapserver {
        url ldap://x.x.x.x/dc=wize,dc=ic?uid?sub?(objectClass=inetOrgPerson);
        binddn "cn=hidded,dc=wize,dc=ic";
        binddn_passwd ;
        #group_attribute member;
        #group_attribute_is_dn on;
        #require group 'cn=docker,ou=groups,dc=example,dc=com';
        require valid_user;
        satisfy all;
    }
```

To build your docker image 
```docker build -t [your-image-name] ./```

Run cointainer inside the instance which was assumed AWS Credential from IAM

```docker run -e REGION=us-west-2 -p 80:80 -p 443:443 -d [your-image-name]```

Run container inside the instance which getting credential from ~/.aws/ folder

```docker run -e REGION=us-west-2 -e AWS_SECRET=... -e AWS_KEY=... -p 80:80 -p 443:443 -d [your-image-name]```

We want to centralize all Docker images at us-west-2, feel free to change the REGION variable if You want to host in another AWS Region

# Mechanism 

These are 3 important files on this container : entrypoint.sh  - renew.sh & auth_update.sh 

Entrypoint will be used to fetch AWS ECR credential from AWS credential where We set as variable or fetching from IAM role

Renew is simple script running every 6 hours to extend our session token between the Proxy with 

AuthUpdate recall from renew script


