---
layout: post
title: "Simulating Redis Unauthorized Access Vulnerability"
excerpt: "If a Redis is publicly accessible and is not protected by password, a remote attacker can exploit this to gain unauthorized access to the server. Let's learn how to set up a vulnerable redis server and attack it. We will also explore how to search and verify vulnerable redis out there with powerful search engine Shodan and automated python exploit"
categories: [pentesting-simulation]
comments: true
image:
  feature: redis-unauthorized-access.png
  show: true
---

Redis, is an open source, widely popular data structure tool that can be used as an in-memory distributed database, message broker or cache. Since it is designed to be accessed inside trusted environments, it should not be exposed on the Internet. However, some Redis' are bind to public interface and even has no password authentication protection. 


Under certain conditions, if Redis runs with the root account, attackers can write an SSH public key file to the root account, directly logging on to the victim server through SSH. This may allow hackers to gain server privileges and delete or steal data, or even lead to an encryption extortion, critically endangering normal business services.


The simplified flow of this process is:
	- Login to a unprotected Redis
	- Change it's backup location to .ssh directory
	- Write the SSH Keys to new backup location
	- Remote connect and login to the target server using SSH key

We should already be familiar how automatic SSH login with private + public keys works.


Now let's get our hands wet and set up a target machine. We'll need two machines, they can be real machines, virtual machines, or remote machines (VPS). As long as the attack end is able to ping the target end, we are good.


Environment setup:
	- Target: redis-3.2.11
	- Attack
