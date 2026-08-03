1. User Management and Permissions

Never use root acc. Create a user account and give him sudo access commands.

edit sudoers file with visudo. sudo visudo.

if no editor selected then, sudo visudo EDITOR=nano

had to add a user in the sudo or wheel section manually if i d onot use usermod -aG sudo username. in visudo go to sudo section and add like:
harry ALL=(ALL: ALL) ALL

username host = )group : permission) forgot the last one.

it will check for syntax error first then confirm changes.

Learned linux basic permissions managing, changing owner of file etc.


2. SSH -Secure access shell

learned how to generate ssh key using ssh-keygen (openssh server) command and take help from man ssh.

ssh-keygen -t ed25519 (fastest and secure algorithim)

learned that if i add my host machines generated key to my remote machine's ~/.ssh/authorized_keys then, I will be able to access it without password. though it can be achieved with script like ssh-copy-id i chose manual method to learn the mechanism.

and if i mess with the ~/.ssh files permission which should be 700 for security and ~/.ssh/authorized_keys fil's permission to 600. giving permission other than that may result in failure to login without password and it may ask me for the password.

and to see the log for ssh service had to use sudo journalctl -u sshd

3. SSH Hardening 

for a secure server first of all i have to change the port which is by default 22 for decreasing the chances of bots trying to access the server using sudo nano /etc/ssh/sshd_config file. 

changing port will not take effect if i do not reload daemon with sudo systemctl reload-daemon and then run the sudo systemctl restart ssh.socket

then i will change PermitRootLogin (standard no) and PasswordAuthentication's value to no. Can find these using search (using slash /) and i have to uncomment them if these are commented with #.

then i shall mandatory check the configuration using sudo sshd -t. no output means perfectly fine. and then restart the service using sudo systemctl restart ssh.

then i triend to login with -o PubKeyAuthentication=no flag to avoid using the ssh key pair and tried to login using password which should have been failed attempt. But surprisingly it logged me in though i disabled the password login to disable in /etc/ssh/sshd_config file. then i found out there is a directory in the /etc/ssh/sshd_config.d/ where there are *.conf file stored for diffrent settings of the main sshd_config file. they overwritten the config and resuliting in avoiding my PasswordAuthentication no setting in the main file. then i had to comment or delete the line and then finally it didn't use my password for login and thus i got my server out of the hackers attack and 100% safe.

4. ufw

this was the last part of topic 1. i learned how to enable firewall to prevent the server from outside attackers. installed with sudo apt install ufw. first of all i needed to see the man ufw for some basics then started the setup.


sudo ufw default deny incoming - I set default incoming to deny.

sudo ufw default allow outgoing - allowed everyoutgoing.

then first allowed the ssh custom port so that i am not locked myself out of my own server. for that i used sudo ufw allow 5259/tcp (5259 was my custom port.)

sudo ufw allow 80/tcp (https port for webserver)
sudo ufw allow 443/tcp (http port for webserver)

notice i allowed only tcp port. if i don't mention the tcp part it allows both tcp and udp which is access permission.as it is standard to give only the required permission i had to ad the slash tcp part. udp for faster speed only. usually not necessary for my kind of service.

then enable the firewall sudo ufw enable and checked the situation using sudo ufw status verbose.

remember first set up those ports then enabled the ufw. if i would have done otherwise i would have kicked myself out of my own server and can not access that without the direct control panel console.

normally anyone out of the internet can know if my server was active pingig my ip. then i disabled it by editing the sudo nano /etc/ufw/before.rules file scrolling down to ok icmp codes for INPUT section find echo-request line and changing Accept to DROP. it will result in timeout when they ping my server.

and i came to know there is a diffrence between deny and reject. rejection let the user know that i am rejecting but deny doesn't let the user know, it just simply ignores the request.

tha's all for topic 1.
