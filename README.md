```
Assignment development worklog:

Intro:
AWS, 3 Instances form a secured loadballanced webserver:

BoxB(nginx)-:8080-
          +----BoxA(haproxy):443---|Internet| 
BoxC(nginx)-:8080-

DOMAIN name:
Created a DOMAIN name kobi-trial.tk from freenom.com and associated it to BoxA(34.242.186.44)

SSL Certificate:
Generated a free SSL certificate with Lets Encrypt:
- Login to the EC2 instance
- git clone https://github.com/certbot/certbot.git
- cd /opt/certbot
- certbot-auto -debug --no-bootstrap
- echo "rsa-key-size = 4096" >> /etc/letsencrypt/config.ini
- echo "email = deepak.singh@solutionanalysts.com" >> /etc/letsencrypt/config.ini
- certbot-auto certonly -d kobi-trial.tk --config /etc/letsencrypt/config.ini --agree-tos
The process generated private key and chain certificate - I had to concatenate the two to one key.

SSHKeys for BoxB ans BoxC:
Generated ssh keypair on the Ansible linux and appended the public key to ~/.ssh/authorized_keys on both EC2 instances.
Also edited /etc/ssh/sshd_config - changed PasswordAuthentication parameter to no

Ansible job:
the playbook runs by:
- ansible-playbook -i ./hosts main.yml -vvv
program outline:

main.yml
     files
        kobi-trial.tk.pem
     roles
        create_nginx
            templates
                nginx.conf
                nginx.init
            tasks
                main.yml
                    nginx.yml
                    hardening.yml
        create_haproxy
            templates
                haproxy.cfg
                haproxy.init
            tasks
                main.yml
                    haproxy.yml
                    hardening.yml
                    
- nginx.yml and haproxy.yml downloads nginx tarball, installs required packages and compiles the code. The copy's config file and init 
  script to the relevant folders.
- hardning.yml files implement the correct iptables rules to restrict access only to the service ports and ssh.
- haproxy SSL certificate is stored in files/ directory and copied to the remote server on deployment.

* When setting IPtables INPUT Policy to DROP, Ansible lost connection. I've commented the step and added it manually after deployment.


```
