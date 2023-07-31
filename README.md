# PROJECT-10
LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

## PROJECT TASK

Implement Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.

- This project consists of two parts:

1. - Configure an Nginx [Nginx](https://www.nginx.com) Load Balancer solution

2. - Register a new domain name and configure secured connection using SSL/TLS certificates

## BACKGROUND KNOWLEDGE

- In Project 9, we achieved load balancer with Apache, aside Apache, NGINX can also be used for load balancer.

- So what is NGINX? Aside Apache, NGNIX is another popular open-source web server software. It is used for reverse proxy, load balancing, and caching. It provides HTTPS server capabilities and is mainly designed for maximum performance and stability. It also functions as a proxy server for email communications protocols, such as IMAP, POP3, and SMTP.

 Some of the differences between __Apache__ and __Nginx__ include:

| Apache | Nginx |
| ---- | --- |
|    Apache runs on all Unix like systems such as Linux, BSD, etc. as well as completely supports Windows.  |Nginx runs on modern Unix like systems; however it has limited support for Windows. |
| Apache uses a multi-threaded approach to process client requests. | Nginx follows an event-driven approach to serve client requests. |
| Apache cannot handle multiple requests concurrently with heavy web traffic. | Nginx can handle multiple client requests concurrently and efficiently with limited hardware resources. |
| Apache processes dynamic content within the web server itself | Nginx can't process dynamic content natively. |
| Apache is designed to be a web server. | Nginx is both a web server and a [proxy server](https://www.fortinet.com/resources/cyberglossary/proxy-server). |
| The performance of Apache for static content is lower than Nginx. | Nginx can simultaneously run thousands of connections of static content two times faster than Apache and uses little less memory.

- It is also extremely important to ensure that connections to your Web solutions are secure and information is [encrypted in transit](https://security.berkeley.edu/data-encryption-transit-guideline) – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

- When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment, there by exposing sensitive information (bank details, social media access credentials, etc.) via non-secured channels. This kind of information security threat is called [Man-In-The-Middle (MIMT) attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

- [SSL and its newer version, TSL](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0,_2.0,_and_3.0) – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

- SSL/TLS uses [digital certificates](https://en.wikipedia.org/wiki/Public_key_certificate) to identify and validate a Website. A browser reads the certificate issued by a [Certificate Authority (CA)](https://en.wikipedia.org/wiki/Certificate_authority) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

- There are different types of SSL/TLS certificates – you can learn more about them [here](https://blog.hubspot.com/marketing/what-is-ssl). You can also watch a tutorial on how SSL works [here](https://www.youtube.com/watch?v=T4Df5_cojAs) or an additional resource [here](https://www.youtube.com/watch?v=SJJmoDZ3il8).

- In this project you will register your website with [LetsEnrcypt](https://letsencrypt.org) Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – [cetrbot](https://certbot.eff.org).

## Setup and technologies used in Project 10
- In this project we will enhance our **Tooling Website** solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

- To simplify, let us implement this solution with **2 Web Servers**, the approach will be the same for 3 and more Web Servers.
- Make sure that you have following servers installed and configured within Project-7:

1. - Two RHEL8 Web Servers

2. - One MySQL DB Server (based on Ubuntu 20.04 LTS)

3. - One RHEL8 NFS server

4. - One **Nginx LB** server(based on Ubuntu 20.04 LTS)

- Your target architecture will look like this:

![10_1](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/39dc9bc2-c2e2-4939-acde-4b4890f099db)   


## A      ***CONFIGURE NGINX AS A LOAD BALANCER***

### 1A. - I created an EC2 VM based on Ubuntu Server 20.04 LTS and name it **Nginx LB** (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections).
  
![10_2](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/e58525a7-9170-4abf-a364-16fb8e9057c9)   

### 2A.   - I had to update **/etc/hosts** file for local DNS with Web Servers’ names (e.g. **Web1 and Web2**) and their local IP addresses, just as it was done on Project 8.
  
 On the NGINX server, execute below...

 `sudo vi /etc/hosts`

 ![10_7](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/b9c44054-45d3-4650-97db-04ca0076d323)
 

### 3A.    - Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

- I had to update the instance and Install Nginx

```
sudo apt update -y
sudo apt install nginx -y
sudo systemctl enable nginx && sudo systemctl start nginx
sudo systemctl status nginx
```

### 4A.     - Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

Copy and paste the following, insert following configuration into http section...

Remember on this line - `server Webserver1 weight=5;`, you have to capture same name used while modifying `sudo vi /etc/hosts` above

```
 upstream myproject {
    server WEB1 weight=5;
    server WEB2 weight=5;
  }

server {
    listen 80;
    server_name ezeonokyproject10.com.ng www.ezeonokyproject10.com.ng;
    location / {
      proxy_pass http://myproject;
    }
  }
```

comment out this line `include /etc/nginx/sites-enabled/*;`
i.e  #include /etc/nginx/sites-enabled/*; in the above file.

Below in green rectangle was added...


![10_8](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/331b58aa-7f62-459c-87d9-8b942a7de775)


We also need to check that our NGINX config file is without errors. So we proceed to run the command below to test the configuration and check for any syntax error

`sudo nginx -t`

![10_9](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/2495cc70-38fd-4f9f-8415-90dd8243ba0b)

Now we need to remove our defualt site so that our reverse proxy will be redirecting to out new configuration file, below was used...

`sudo rm -f /etc/ngninx/sites-enabled/defaults`


Link the `/etc/nginx/nginx.conf` to the `/etc/nginx/sites-enabled`
Now we return to our NGINX site enabled...the site enabled is empty, we target to link our load balancer config file that we just created in our site available to our site enabled so that the NGINX can access our configuration through it


```
sudo rm -rf /etc/nginx/sites-enabled/defaults

sudo ln -s /etc/nginx/nginx.conf /etc/nginx/sites-enabled
```

After the link is made, change directory to  cd /etc/nginx/sites-enabled(note previously, this directory was empty) and confirm the link was successful... see below

![10_11](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/09d4afbd-99ec-4ffd-a835-6717fdfd444d)


Restart Nginx and make sure the service is up and running

```
sudo systemctl reload nginx
sudo systemctl status nginx
```

![10_10](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/318eef94-89df-4aa3-9248-ffe0b83630c3)


- **Side Self Study**: Read more about HTTP load balancing methods and features supported by Nginx [on this page](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/).

Continue to step B below....


## B     ***REGISTER A NEW DOMAIN NAME AND CONNECT IT TO YOUR ROUTE 53 ON THE AWS LOGIN***

### 1B.     Register a Domain Name. Connect your Route 53 hosted zone to your newly registered domain
- First, we need to register a new domain name. We can do it using any Domain name registrar – a company that manages reservation of domain names. godaddy.com etc
  
- I choose https://www.domainking.ng/ for my domain name registration. Domain Name - ezeonokyproject10.com.ng was registered.

- On the AWS console, i searched for route 53, clicked on it and create a hosted zone. I Created the hosted zone by filling in the required details of our new domain name.
  
![10_3](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/03df2fb8-6325-43f7-b0db-22059eadfaae)

For our route 53 to connect to our domain name, we need to copy each of our nameservers from the route 53 to the new domain name on the www.domainking.ng

Copy these details under `Value/Route Traffic To`

![10_5](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/699b1928-9236-4ae6-ae42-c806a772396b)

 ....to below on the newly registered domain page, and then click  on "change nameservers", as per step 12 below.

 ![10_6](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/90d2b002-e6eb-4305-abcc-9bbce4b7c87c)

 So our route 53 is now connected to our new domain name -  ezeonokyproject10.com.ng 


 ![10_12](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/f28dee11-dd56-447a-bf94-8b5045458391)




### 2B.     - Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

When you launch an EC2 instance, you receive a Public IP address by which that instance is reachable from the internet. Once the instance is stoped and restarted, the instance gets a new Public IP for the same instance. So it's basically a problem to connect your instance from internet for not having a static IP. To overcome this problem, we attach an Elastic IP to an Instance. Below is a step guide on how to generate an elastic IP and associate it with a running instance.


![10_64](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/b9b58488-0e01-4d57-a729-42d62f781d85)

`How to create an elastic IP and associate it to our running instance`


### 3B.      - Update DNS A Record on our AWS Route 53
Now we proceed to update [A record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) in the registrar of our Route 53 to point to Nginx LB using static IP address

We go back to the route 53 and click "create record" using the NGNIX LB Static IP.

We will create [A record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) for www.ezeonokyproject10.com.ng and ezeonokyproject10.com.ng using the same static IP.

![10_61](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/79512134-f1f6-4856-97aa-23902b56fecf)


![10_62](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/64a7c5ae-1a6c-4936-808d-40968dcb75b2)


This will allow us login using any of the  domain name format - www.ezeonokyproject10.com.ng or ezeonokyproject10.com.ng

![10_63](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/97889352-46f2-496a-97af-0f7bcf477dc7)



Learn more on how to associate your domain name to your Elastic IP [on this page](https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233).

- **Side Self Study**: Read about different [DNS record types](https://www.cloudflare.com/learning/dns/dns-records/) and learn what they are used for.

### 4B.   - On your Browser, try Loading NGINX-LB using the new domain name 

We then try to access the nginx-LB from the browser using our domain name - www.ezeonokyproject10.com.ng or ezeonokyproject10.com.ng

Below error was encounter...

![10_13a](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/d9ad49e1-9843-4655-83c3-b35bddc9df3b)

A ping to the site was done, using `ping -c 2 www.ezeonokyproject10.com.ng` . Error UNKNOWN HOST was returned

Also DNS check was done, https://dnschecker.org/#A/ezeonokyproject10.com.ng, no DNS record was seen.

The NS record on AWS Route 53 was checked and confirmed OK, same NS record was duely updated on the on my domain registar(https://clients.domainking.ng/)


I awaited for a while before above and below was able to load, It appear my Domain Registar did not immediately update my new IP address, hence the earlier received error message when i tried loading the site. Some Data Centre had not captured the the new IP, possibly the IP address was still propagating, hence the delay in loading of below site.

![10_13b](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/eaa8e6f3-cd93-41c3-bd41-f63118b34a07)

 Notice my domain registrar has not fully update my DNS record on some data centre. Several data centres haven’t updated my IP to my domain name. Anyone  from such region trying to access my site will isuues accessing my domain name.


Confirmation of successful site loading

![10_13](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/b1d7ebd8-a1cd-4ede-bd25-2457de3456fb)


In the above we will see that the connection is not secured. To configure a secure connection using SSL/TLS, we use below...


 
## C       ***CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES ON YOUR REGISTERED DOMAIN NAME***

- Let us make necessary configurations to make connections to our Tooling Web Solution secured!

### 1C.      - Install certbot and request for an SSL/TLS certificate

Install certbot and request for an SSL/TLS certificate `sudo apt systemctl certbot -y`, also Install the dependencies `sudo apt install python3-certbot-nginx -y`

```
sudo apt systemctl certbot -y
sudo apt install python3-certbot-nginx -y
```

Request a certificate by following the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from `nginx.conf`

`sudo certbot --nginx`

A running above, below error was encountered...

![10_13c](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/f6affec4-abc4-48e1-808f-2462297a25e0)

After changes the the NGINX config file, below was executed

`sudo certbot --nginx -d ezeonokyproject10.com.ng -d www.ezeonokyproject10.com.ng`

![10_13d](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/444d9f02-28b9-4e4e-87d6-f78bf799ef34)


Test secured access to your Web Solution by trying to reach https://www.ezeonokyproject10.com.ng

![10_13e](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/6dfbea18-fc2f-4595-b7e6-005cd4ecaa11)

We should be able to access our website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s address bar. When we click on the padlock icon, we can see the details of the certificate issued for our website.


### 2C.        - Set up periodical renewal of your SSL/TLS certificate.

By default, LetsEncrypt certificate is valid every 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode.

`sudo certbot renew --dry-run`

![10_13f](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/93b487dd-f67c-43da-abb8-95d96e07a5d4)

Best pracice is to have a scheduled job to run renew command periodically.

Let us configure a cronjob to run the renewal twice per day. The command checks to see if the certificate on the server will expire within the next 30 days, and renews it if so.

To do so, lets edit the crontab file with the following command:

```
crontab -e

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

![10_13f](https://github.com/EzeOnoky/Project-Base-Learning-10/assets/122687798/9c08f9b7-8d61-411d-9680-d342fa0c1718)


# CONGRATULATIONS EZE !
# - I have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.

- Side Self Study: Refresh your cron configuration knowledge by watching [this video](https://www.youtube.com/watch?v=4g1i0ylvx3A)

- You can also use this handy [online cron expression editor](https://crontab.guru).







