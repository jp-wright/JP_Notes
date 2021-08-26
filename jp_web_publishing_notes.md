## JP Website Notes


1. Register a domain name at [godaddy.com](godaddy.com) or similar place.
2. Get web hosting services at a place like [digitalocean.com](digitalocean.com).
3. Access the server through your terminal and publish updates, etc.



### 1. Register Domain Name
Using [godaddy.com](godaddy.com), search for your desired domain name.  There will be many options for domain endings, such as ".com", ".us", ".net", ".org", etc.  They each have general connotations.
- **.com**: commercial website, either representing a business or offering services
- **.net**: originally meant for a network of computers, now an alternative to .com
- **.us**: originally meant for personal or non-profit us in the USA only; most countries have their own domain suffix, such as .uk for United Kingdom or .de for Germany (Deutschland)
- **.org**: for non-profit organizations
- **.edu**: restricted to accredited degree-awarding schools (at least in USA)
- **.gov**: USA governmental agencies
- **.mil**: USA military agencies

A domain name is the URL 'name' we think of for a given website, such as [google.com](google.com), [fivethirtyeight.com](fivethirtyeight.com), [colorado.edu](colorado.edu), or [nasa.gov](nasa.gov).  It's like the personalized name plate on your mailbox that tells people what to call the residence.  It is not, however, the actual address itself.  In web hosting, we call that the Internet Protocol (IP) address and we'll get to that in step two.

For example, my first test case was using [pizefootball.us](pizefootball.us).  

GoDaddy said it was available (shocking, I know), so I purchased it for 1 year at $7.99.  Optional services GoDaddy offered include "Domain Ownership Protection," (which claims to prevent your domain being surreptitiously stolen from you), a website building template utility that starts free but incurs a $9.99 charge after a given period, and the option to create an email address using the newly purchased domain name, with different price tiers for different amounts of email storage ($1.99/month - $8.99/month).  Depending on your use case, I think the email option is smart and simple.

__Note__: the "\.us" domain has certain legal restrictions on its intended use which make it cheaper than ".com" or others.  That's why I chose it for my test case.

Once registered, look up the newly purchased domain under "Domains" in your account.  Locate "Manage DNS" in your domain's "additional settings" area.  ![DNS Settings](/Users/jpw/Dropbox/Data_Science/jp_notes/markdown_notes_repo/images/web_publishing/godaddy_dns.png)

The important row as far as we are concerned is the top one, where we see **Type: A**.  

The default **Value** of row **A** is *Parked*, which means this domain name doesn't point to anywhere (it has no anchor).  Once we get an IP address via our web host, we will paste it into the cell that has *Parked* and our domain name will point to the new IP address (i.e. website) we've registered.


### 2. Get an IP address from a Web Host
With Terry's recommendation, I used [digitalocean.com](digitalocean.com), known for fast performance and ease of modification.  Their prices are very reasonable.  They call their virtual machines "Droplets."  They offer plenty of advanced setups which are beyond my knowledge or current use case, including Kubernetes clusters and load balancers.  In general Digital Ocean is considered to be good for small-to-medium sized hosting, while AWS is best for large scale apps that might require heavy customization.

Once you register a new account with Digital Ocean (note: you might have to pay a $5 pre-pay fee which will be applied to future costs.  It acts as an anti-spam measure.), you will need create a new *Droplet* (virtual machine).  Following registration, you might be on the "Marketplace" page and if so, you can click on the "LAMP" Droplet framework, or in the top right of the page (or perhaps elsewhere depending on the exact page you see -- it was a bit confusing for me), you can click "Switch To" and then "Control Panel."

Once in Control Panel, choose "Create" then "Droplet."  
Here you will have many options and can feel overwhelmed if you're new to this, as I did. I wanted something simple and standard, and my requirements are *very* pedestrian.    

First, in the "Marketplace" tab, I selected the following for my Droplet:

1. **Choose an Image**: LAMP on Ubuntu (20.04). LAMP = Linux (operating system), Apache (HTTP Server), MySQL (database), PHP (or Perl or Python - programming languages).  This is a very common web stack.
2. **Choose a CPU Plan**: There are various configuration options that scale with demand, here, and are priced accordingly, from $5/month to $2,480/month.
   - Basic CPU (shared) plan -- don't need more unless getting a lot of traffic
   - I chose the absolute cheapest CPU setup: $5/month for 1 GB CPU, 25 GB SSD, and 1000 GB Transfer, all using "Regular Intel" chips.  
3. **Add Block Storage**: You can "add block storage" if you need more.  I did not.
4. **Choose a datacenter location**: One on either the USA East Coast or West Coast, one in Canada, a few in Asia and a few in Europe.  Pick the one closest to you.
6. **Authentication**: Choose between SSH Key (more secure, one-time login) or a traditional Password (less secure, have to re-enter it when logging in).
   - You should use an SSH key.  It's the better choice in both security and convenience.
   - If you are unsure about how to use an SSH key, **see below**.  It is very simple.
   - Once created, select any or all SSH keys you've saved to this account.  It is okay to have more than one SSH key for a given Droplet.
7. **Choose a Hostname**: This will be the name of your computer as far as your website is concerned.  It is not public facing, but it is displayed in your terminal as the root environment when you're logged into the server.  As such, a short and descriptive name is best.  Something like `jp-server` or `jp-macbook`, etc.
8. **Enable backups**: An optional service you can get for $1/month. Digital Ocean says, "A system-level backup is taken once a week, and each backup is retained for 4 weeks."  I chose to enable backups, for such a cheap price.

Once done, complete the creation process and you'll be given an IP address.  It might take up to 5-10 minutes, but is usually faster than that.


Here's the IP Address I was given: **`198.211.98.5`**  




#### Creating and Using a New SSH Key
Open your terminal.  
Type `cd ~/.ssh` to enter the hidden ssh directory.
Type `ls -a` to see what files you have already.  For SSH keys, we expect to have a pair of files for each key, one being *public* and the other *private*, such as `id_rsa.pub` and `id_rsa`.  The `.pub` file is a public hashed version of the private (non-`.pub`) file.  We use the public file whenever a site or server wants our SSH key.  Never use the private file.  

If you have an SSH key pair, open the `.pub` file by typing `more id_rsa.pub`.  You'll be shown your full public SSH key, and it will look something like this:

```bash
ssh-rsa EAAAADAQABAAABAQCvmHO7NeyePnd/t/L7kauGYgSUihHR32IHD/
ZRMWpY0F0thSquaG0UUda2A2o8V13WMiKpVi5Z0U19Lfu2NQ/kqXYryjyw2VI4Svs/
2YavULUpttN8dLM2pc5LXHScGXJEjMDKvnRjBsjZXbEcBXDJu+iSywgXC0QAR5279KnV23Ehn5M4CW
AxyP2TO7Qm9 jpw@JPW-rMBP.local
```

(Note this is not a real public SSH key.  Most sites will tell you whether the key you've entered is valid or not as soon as you paste it).

Some terminal windows will cut off part of the SSH key when you view it to copy, and using certain in-terminal editors, like `nano` can cause this more often, I think.  As such, you can use the command `cat ~/.ssh/id_rsa.pub` to copy the full SSH key if needed.

If you don't have an `id_rsa.pub` file, we will need to create a new SSH key.  It's easy.  

Type `ssh-keygen` in the terminal.  Next you will be asked to create and confirm a passphrase for the key.  While all security experts suggest it is highly recommended, and they're right, it does mean you have to keep up with the password you create for this SSH key or it will not be accessible for the various apps or services you've used it with in the past.  You can simply hit `Enter` to enter a blank password, which means there will not be one attached to this SSH key.

```bash
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

**Note**: You can create different types of SSH keys -- not just the standard RSA key.  For example, use `ssh-keygen -t ed25519 -C <your email address here>`

You'll be shown "randomart" that identifies your SSH key.  I've never heard of it being used.

Now, using the instructions above, open your newly-created public SSH key, copy it, and paste into the SSH Key box on Digital Ocean.  Name it something clear, like "jp_ssh_ed25519" or "jp_macbook_ssh."

Back on the Control Panel configuration page for your Droplet, click the "Add public SSH Key" button and paste

Here's a link for more detailed info on [SSH keys and using them with Digital Ocean](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/).





### Connect Domain Name to IP Address
We've registered a new name for our website (`pizefootball.us`, the domain name).  We've ordered a new house for our website to live in (`198.211.98.5`, IP address).  Now we have to connect the two, so that people can find our website by its name instead of by its numerical IP address.  

In Go Daddy, in your "Manage DNS" settings, change the **Value** of row **A** from `Parked` to the new IP Address, `198.211.98.5` in this case.

Now any time someone searches for `pizefootball.us`, our new website will load.  They do not have to use the IP address at all.  This means we now have a functional website backend that is waiting for someone to develop a functional front end for.


### Access Web Server in Terminal
Access the server from the terminal via `ssh root@pizefootball.us` or `ssh root@198.211.98.5`.  Since we've pointed our domain name to our new IP address, we can access the website using either its 'name' or 'address.'

After creation, if you try to access it too soon before it's been fully processed by Digital Ocean, you'll see some messages similar to:

```
Please wait while we get your droplet ready...
Connection to 198.211.98.5 closed.
```

You might also have to add the new IP Address to what amounts to your address book of known hosts.  You'll see something like the following:

```
The authenticity of host '198.211.98.5 (198.211.98.5)' can't be established.
ECDSA key fingerprint is SHA256:7ODltyT7hsrNSj3djFyL5wbZxLc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '198.211.98.5' (ECDSA) to the list of known hosts.
```

Choose "yes" to add it.



## Further LAMP Information - Settings, Certificates, and APIs
Here's [Digital Ocean's LMAP guide](https://marketplace.digitalocean.com/apps/lamp#getting-started) which details the software installed as well as settings, how to use certificates for security, and developing APIs for your site.

![LAMP Software](/Users/jpw/Dropbox/Data_Science/jp_notes/markdown_notes_repo/images/web_publishing/LAMP_software.png)



Here are some key points from the guide.

> In addition to the package installation, the 1-Click also:
>
> + Enables the UFW firewall to allow only SSH (port 22, rate limited), HTTP (port 80), and HTTPS (port 443) access.
> + Sets the MySQL root password and runs mysql_secure_installation.
> + Sets up the `debian-sys-maint` user in MySQL so the system's init scripts for MySQL will work without requiring the MySQL root user password.

> After you create a LAMP One-Click Droplet:
>
> + You can view the LAMP instance immediately by visiting the Droplet's IP address in your browser.
> + You can log into the Droplet as root using either the password emailed to you or with an SSH key, if you added one during creation.
> + The MySQL root password is in `/root/.digitalocean_password`.
> + The web root is `/var/www/html`.
> + You can get information about the PHP installation by logging into the Droplet and running `php -i`.

For now, I want to highlight `/var/www/html`.  

This is where we get into root on the web server.  In terminal, type `cd /var/www/html` to go there.  The homepage of your website will be whatever is in a file called `index.html`, by default.  You can use an "index" file of any type, so long as the server has the right software installed to read it.  For example, if you build a dashboard web app in Plotly's *Dash* platform using Python, you could create a file called `index.py` that runs your Dash app.  Obviously, you'd need Python installed on the server to run it.

You can update your website by creating new HTML (or whatever type) files on your local and copying / uploading them to the server.  You can create your own file directory structure on the server which your pages will navigate. All your typical commands in the terminal apply since it is a Linux based server.  Using `ls -a` will list all the files in the working directory, etc.


To move a file to a holding location, we rename its extension to `.hold` as a quick fix.

`mv index.html html.hold`

We can then edit it quickly with `nano`.
`nano index.html`

Of course, we would not develop an actual site this way, just make quick changes on the server (which itself can be risky if not done carefully).


##### Streaming Info
Type `ping pizefootball.us` (or `ping 198.211.98.5`) to get streaming info on server usage and response time.

```
64 bytes from 198.211.98.5: icmp_seq=0 ttl=50 time=34.198 ms
64 bytes from 198.211.98.5: icmp_seq=1 ttl=50 time=25.659 ms
64 bytes from 198.211.98.5: icmp_seq=2 ttl=50 time=24.930 ms
```


### Package Manager - APT
The package manager on the server is called APT.  The best first step in general website performance and security is to ensure your packages are updated.

Once you've ssh'ed into the server in terminal, type `apt update` to see the list of packages that can be updated.  You'll get a lot of streaming output and finally receive this message once completed.

> Reading package lists... Done  
> Building dependency tree  
> Reading state information... Done  
> 10 packages can be upgraded. Run `apt list --upgradable` to see them.  

Then type `apt upgrade -y` to upgrade those packages with a pre-emptive "yes" to any prompts asking if you want to upgrade.  Per usual, you'll get a lot of streaming output about package updates.




### Uncomplicated Firewall (UFW)
Type `ufw status` to see the allowed ports and any respective limits they have.  

```
To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere
Apache Full                ALLOW       Anywhere
22/tcp (v6)                LIMIT       Anywhere (v6)
Apache Full (v6)           ALLOW       Anywhere (v6)
```

UFW stands for "Uncomplicated Firewall", which is a default feature of Ubuntu.  Read more [here](https://en.wikipedia.org/wiki/Uncomplicated_Firewall).


### HTTPS
Digital Ocean includes the ability to enforce HTTPS connection protocols on your site.
>  Certbot is preinstalled. Run it to configure HTTPS. See
   https://do.co/3gY97ha#enable-https for more detail.

I haven't done this yet, but it is standard practice and recommended.  Here's Digital Ocean's how-to steps for it.

> In addition, there are a few customized setup steps that we recommend you take.
>
> Creating an Apache virtual hosts file for each site maintains the default configuration as the fallback, as intended, and makes it easier to manage changes when hosting multiple sites.
>
> To do so, you'll need to create two things for each domain: a new directory in /var/www for that domain's content, and a new virtual host file in /etc/apache2/sites-available for that domain's configuration. For a detailed walkthrough, you can follow [How to Set Up Apache Virtual Hosts](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-16-04).
>
> Setting up an SSL certificate enables HTTPS on the web server, which secures the traffic between the server and the clients connecting to it. Certbot is a free and automated way to set up SSL certificates on a server. It's included as part of the LAMP One-Click to make securing the Droplet easier.
>
> To use Certbot, you'll need a registered domain name and two DNS records:
>
> + An A record from a domain (e.g., example.com) to the server's IP address
> + An A record from a domain prefaced with www (e.g., www.example.com) to the server's IP address
>
> Additionally, if you're using a virtual hosts file, you'll need to make sure the server name directive in the VirtualHost block (e.g., `ServerName` example.com) is correctly set to the domain.
>
> Once the DNS records and, optionally, the virtual hosts files are set up, you can generate the SSL certificate. Make sure to substitute the domain in the command.
>
>```
> certbot --apache -d example.com -d www.example.com
>```
>
> HTTPS traffic on port 443 is already allowed through the firewall. After you set up HTTPS, you can optionally deny HTTP traffic on port 80:
>
>```
> ufw delete allow 80/tcp
>```
>
> For a more detailed walkthrough, you can follow [How to Secure Apache with Let's Encrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04) or view [Certbot's official documentation](https://certbot.eff.org/docs/using.html).
>
> You can serve files from the web server by adding them to the web root (`/var/www/html`) using [SFTP](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server) or other tools.
>
> A newly-created LAMP Droplet includes an `index.html` web page. You can change this by uploading a custom `index.html` file or remove it.

<BR>
<BR>

### API Creation
More from Digital Ocean's guide:

> In addition to creating a Droplet from the LAMP 1-Click App via the control panel, you can also use the [DigitalOcean API](https://digitalocean.com/docs/api).
>
> As an example, to create a 4GB LAMP Droplet in the SFO2 region, you can use the following curl command. You'll need to either save your [API access token](https://docs.digitalocean.com/reference/api/create-personal-access-token/) to an environment variable or substitute it into the command below.
>
>```bash
> curl -X POST -H 'Content-Type: application/json' \
>      -H 'Authorization: Bearer '$TOKEN'' -d \
>     '{"name":"choose_a_name","region":"sfo2","size":"s-2vcpu-4gb","image":"lamp-20-04"}' \
>     "https://api.digitalocean.com/v2/droplets"
>```
