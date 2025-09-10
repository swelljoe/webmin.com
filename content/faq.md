---
title: "FAQs"
summary: "Frequently Asked Questions"
author: "Jamie Cameron"
tags: ["faq"]
date: 2025-07-11
showtoc: true
weight: 3
---

> #### I think I have found a bug in Webmin
First of all, try upgrading to the latest version. Many bugs in older releases are fixed in the latest version.

The second place to check is the [Webmin changelog](/changelog) page, on which bug-fixes for the current version are posted. The easiest way to install all the latest updates is via package manager by setting up [Webmin repository](/download#setup) or is to use **Webmin Configuration** module.

If you really have found a new bug, go to the Webmin GitHub repository to [submit a new](https://github.com/webmin/webmin/issues) issue report.

---

> #### What effect will Webmin have on my existing configuration files?
Just installing Webmin will not cause any config file changes to be made. When you start to use it, only the config files related to the changes that you make in Webmin will be modified. For example, using the Apache Webserver module would not effect your Postfix configuration.

---

> #### How do I install Webmin if port 10000 is already in use?
If port 10000 is already in use and you want to install Webmin using a package
manager, you can specify a different port by setting the `WEBMIN_PORT` environment
variable alongside with running the installation command. For example, to
install Webmin on port 15000 on Debian and derivatives, use the following command:

```text
WEBMIN_PORT=15000 dpkg -i webmin_2.202_all.deb
```

On RHEL and derivatives:
```text
WEBMIN_PORT=15000 rpm -i webmin-2.202-1.noarch.rpm
```
---

> #### How do I change my Webmin password if I can't login?
If you installed Webmin using package manager (i.e. _rpm_ or _deb_) use the following command to change Webmin user password:
```text
webmin passwd username
```

If not, use the same command found in `bin/` sub-directory of Webmin main installation, i.e. under `/usr/libexec/webmin` or `/usr/share/webmin` or `/usr/local/webmin` directory.

---

> #### Can I run Webmin or Usermin behind reverse proxy?
Yes, this can be done with some configuration.
{{< details-start post-content-details "<strong>Apache</strong>" >}}

If you just want Webmin to be accessible via Apache gateway follow the steps below:
 
 - Edit `/etc/webmin/config` file and add the following line:
 ```
 referers=webmin.example.com
 ```
 - Edit `/etc/webmin/miniserv.conf` file, add the following lines and restart Webmin afterwards by calling `/etc/webmin/restart` command:
 ```text
  redirect_ssl=1
  redirect_host=webmin.example.com
 ```
 - Edit `/etc/webmin/xterm/config` file and add the following line:
 ```
 host=webmin.example.com
 ```
 - Enable `mod_proxy` and `mod_proxy_wstunnel` for your Apache webserver
 - Create a `VirtualHost` block with the following directives to the Apache configuration, and restart Apache afterwards. Remember to replace `VirtualHost` IP address, `ServerName` and SSL certificates paths with your own:
 ```
 <VirtualHost 1.2.3.4:443>
    ServerName webmin.example.com

    # Enable the usage of the SSL/TLS protocol engines
    SSLEngine on
    SSLProxyEngine on

    # Point to files with SSL certificates for virtual host
    SSLCertificateFile /etc/ssl/domains/example.com/ssl.combined
    SSLCertificateKeyFile /etc/ssl/domains/example.com/ssl.key
    
    # Use only secure version of the TLS protocol (TLSv1.3)
    SSLProtocol         all -SSLv3 -TLSv1 -TLSv1.1 -TLSv1.2
    SSLHonorCipherOrder off
    SSLSessionTickets   off

    # Disables the remote server certificate checks
    # (only needed for self-signed certificates)
    SSLProxyCheckPeerCN     off
    SSLProxyCheckPeerName   off
    SSLProxyCheckPeerExpire off

    # Disable proxying for all /.well-known requests. It will 
    # only be useful, if a domain has "DocumentRoot" defined
    ProxyPass /.well-known !

    # Proxying both HTTP and websockets at the same time,
    # where the websockets URL's are not websocket-only
    # or not known in advance
    ProxyPass / https://localhost:10000/
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "wss://localhost:10000/$1" [P,L]
</VirtualHost>
 ```
Now all requests to `webmin.example.com` to the Apache virtual host will then be passed through to the Webmin server on `localhost` port `10000`.

{{< alert warning question "Want to run under Apache sub-directory?" "Follow the instructions below instead." >}}

- Edit `/etc/webmin/config` file and add the directives mentioned below:
```text
referers=webmin.example.com
webprefix=/webmin
webprefixnoredir=1
```
- Edit `/etc/webmin/miniserv.conf` file, add directives mentioned below and restart Webmin afterwards by calling `/etc/webmin/restart` command:
```
redirect_prefix=/webmin
cookiepath=/webmin
```
- Enable `mod_proxy` and `mod_proxy_wstunnel` for your Apache webserver
- Create a `VirtualHost` block with the following directives to the Apache configuration, and restart Apache afterwards. Remember to replace `VirtualHost` IP address, `ServerName` and SSL certificates paths with your own:
```
 <VirtualHost 1.2.3.4:443>
    ServerName webmin.example.com

    # Enable the usage of the SSL/TLS protocol engines
    SSLEngine on
    SSLProxyEngine on

    # Point to files with SSL certificates for virtual host
    SSLCertificateFile /etc/ssl/domains/example.com/ssl.combined
    SSLCertificateKeyFile /etc/ssl/domains/example.com/ssl.key
    
    # Use only secure version of the TLS protocol (TLSv1.3)
    SSLProtocol         all -SSLv3 -TLSv1 -TLSv1.1 -TLSv1.2
    SSLHonorCipherOrder off
    SSLSessionTickets   off

    # Disables the remote server certificate checks
    # (only needed for self-signed certificates)
    SSLProxyCheckPeerCN     off
    SSLProxyCheckPeerName   off
    SSLProxyCheckPeerExpire off

    # Disable proxying for all /.well-known requests. It will 
    # only be useful, if a domain has "DocumentRoot" defined
    ProxyPass /.well-known !

    # Proxying both HTTP and websockets at the same time,
    # where the websockets URL's are not websocket-only
    # or not known in advance
    ProxyPass /webmin/ https://localhost:10000/
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/webmin/?(.*) "wss://localhost:10000/$1" [P,L]
</VirtualHost>
```

Now all requests to `webmin.example.com/webmin/` (pay attention to the trailing slash) to the Apache virtual host will then be passed through to the Webmin server on `localhost` port `10000`.

{{< alert primary exclamation "Usermin via Apache gateway?" "The instructions are exactly the same as for Webmin with the only difference that Usermin default port is `20000` and configuration files are located in `/etc/usermin` directory." >}}
{{< alert danger exclamation-triangle "Getting permission denied error?" "This could be due to SELinux restrictions. Check the SELinux configuration for web server network connections." >}}

{{< details-end >}}
{{< details-start post-content-details "<strong>Nginx</strong>" >}}

If you just want Webmin to be accessible via Nginx reverse proxy follow the steps below:
 
 - Edit `/etc/webmin/config` file and add the following line:
 ```
 referers=webmin.example.com
 ```
 - Edit `/etc/webmin/miniserv.conf` file, add the following lines and restart Webmin afterwards by calling `/etc/webmin/restart` command:
 ```text
  redirect_ssl=1
  redirect_host=webmin.example.com
 ```
 - Create a `Server` block with the following directives to the Nginx configuration, and restart Nginx afterwards. Remember to replace `server_name`, `listen` IP address and SSL certificates paths with your own:
 ```
 server {
    server_name webmin.example.com;

    # Enable SSL/TLS and HTTP2
    listen 192.168.50.119:443 ssl http2;

    # Point to files with SSL certificates for virtual host
    ssl_certificate /etc/ssl/domains/example.com/ssl.cert;
    ssl_certificate_key /etc/ssl/domains/example.com/ssl.key;

    # Use only secure version of the TLS protocol (TLSv1.3)
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # Disable proxying for all /.well-known requests. It will 
    # only be useful, if a domain has "root" defined
    location ^~ /.well-known/ {
        try_files $uri /;
    }

    # Proxying both HTTP and websockets
    location / {
        proxy_pass https://localhost:10000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection Upgrade;
        proxy_set_header Host $host;

        # Disable buffering to make progressive
        # output work as expected
        proxy_buffering off;
        proxy_request_buffering off;

        # Enable large file uploads
        client_max_body_size 64g;
    }
}
 ```
Now all requests to `webmin.example.com` to the Nginx virtual server will then be passed through to the Webmin server on `localhost` port `10000`.

{{< alert warning question "Want to run under Nginx sub-directory?" "Follow the instructions below instead." >}}

- Edit `/etc/webmin/config` file and add the directives mentioned below:
```text
referers=webmin.example.com
webprefix=/webmin
webprefixnoredir=1
```
- Edit`/etc/webmin/miniserv.conf` file, add the directives mentioned below and restart Webmin afterwards by calling `/etc/webmin/restart` command:
```
redirect_ssl=1
redirect_host=webmin.example.com
redirect_prefix=/webmin
cookiepath=/webmin
```
- Edit `/etc/webmin/xterm/config` file and replace previously added `host` directive with the the following:
```
host=webmin.example.com
```
- Create a `Server` block with the following directives to the Nginx configuration, and restart Nginx afterwards. Remember to replace `server_name`, `listen` IP address and SSL certificates paths with your own:
 ```
 server {
    server_name webmin.example.com;

    # Enable SSL/TLS and HTTP2
    listen 192.168.50.119:443 ssl http2;

    # Point to files with SSL certificates for virtual host
    ssl_certificate /etc/ssl/domains/example.com/ssl.cert;
    ssl_certificate_key /etc/ssl/domains/example.com/ssl.key;

    # Use only secure version of the TLS protocol (TLSv1.3)
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # Disable proxying for all /.well-known requests. It will 
    # only be useful, if a domain has "root" defined
    location ^~ /.well-known/ {
        try_files $uri /;
    }

    # Proxying both HTTP and websockets
    location /webmin/ {
        proxy_pass https://localhost:10000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection Upgrade;
        proxy_set_header Host $host;

        # Disable buffering to make progressive
        # output work as expected
        proxy_buffering off;
        proxy_request_buffering off;

        # Enable large file uploads
        client_max_body_size 64g;
    }
}
 ```

Now all requests to `webmin.example.com/webmin/` to the Nginx virtual host will then be passed through to the Webmin server on `localhost` port `10000`.

{{< alert primary exclamation "Usermin via Nginx reverse proxy?" "The instructions are exactly the same as for Webmin with the only difference that Usermin default port is `20000` and configuration files are located in `/etc/usermin` directory." >}}
{{< alert danger exclamation-triangle "Getting permission denied error?" "This could be due to SELinux restrictions. Check the SELinux configuration for web server network connections." >}}

{{< details-end >}}

---
> #### How to set up Cloudflare Tunnel to work properly with Webmin?

* Add `referers=your.domain.tld` to `/etc/webmin/config` file
* Add `redirect_host=your.domain.tld` to `/etc/webmin/miniserv.conf` file
* Set up your Cloudflare Tunnel for Webmin with the following configuration:
    ```yml
    tunnel: 00000000-1111-222-3333-444444444444
    credentials-file: /path/to/cloudflared/00000000-1111-222-3333-444444444444.json

    ingress:
      - hostname: your.domain.tld
        service: https://127.0.0.1:10000
        originRequest:
          noTLSVerify: true
          httpHostHeader: your.domain.tld

      - service: http_status:404
    ```
 * Restart Webmin with `/etc/webmin/restart` command

---

> #### My browser reports _Document contains no data_ after turning on SSL
If you are using SSL, make sure you connect to a URL like `https://myhost:10000/` instead of `http://myhost:10000/`. Without the https, your browser won't use SSL mode and thus will display this error.

---

> #### How do I run `setup.sh` script?
After extracting the Webmin _tar_ file, `cd` into the `webmin-current` directory and type `./setup.sh`. Because the _root_ user on many system does not have the current directory in his path, just typing `setup.sh` will not work.

---

> #### How do I install new modules?
Once you have downloaded a new module as a `.wbm` file, enter the **Webmin Configuration** module and click on the **Webmin Modules** button. Then use the form at the top of the page to install the module either from the local filesystem of the server Webmin is running on, or uploaded from the client your browser is on.

---

> #### How do I install Perl on systems that do not have it as standard?
You need to [download](http://www.cpan.org/src/stable.tar.gz) and compile the latest Perl from source.

---

> #### How do I log Webmin actions and the files they have changed?
By default, basic logging is enabled in Webmin. To turn on full logging, go into the **Webmin Configuration** module, click on the **Logging** icon and turn on the **Log changes made to files by each action** option. This will record all file changes and commands run by Webmin. Once logging is enabled, all actions performed from then on can be viewed in the **Webmin Actions Log** module.

---

> #### When I download the `.tar.gz` version of Webmin, why do I get a `.tar` file? Or why is the file so much larger than the size shown on the download page?
Your browser has automatically gunzipped the file for you. Just rename it to `webmin-current.tar` (if it hasn't been already) and skip the gunzip step in the install instructions.

---

> #### My browser complains about the Webmin certificate when in SSL mode
This happens because the default SSL certificate that is generated by Webmin is not issued by a recognized certificate authority. From a security point of view, this makes the certificate less secure because an attacker could theoretically redirect traffic from your server to another machine without you knowing, which is normally impossible if using a proper SSL certificate. Network traffic is still encrypted though, so you are safe against attackers who are just listening in on your network connection.
 
If you want to be really sure that the Webmin server you are connecting to is really your own, the only solution is to request a certificate from an authority like [Let's Encrypt](https://letsencrypt.org/) (can be done for free using Webmin) or purchase it from companies like GoDaddy or Comodo. That certificate is associated with your server's hostname and will be recognized by web browsers.

---

> #### In the _Users and Groups_ module, how can a script that run with _Before and after commands_ access environment variables?
The follow environment variables are set by Webmin before the script is called :
  * `$USERADMIN_ACTION` - This can be set to `CREATE_USER`, `MODIFY_USER`, `DELETE_USER`, `CREATE_GROUP`, `MODIFY_GROUP` or `DELETE_GROUP` depending on what was just done.
  * `$USERADMIN_USER` - The username of the Unix user who was just created, modified or deleted.
  * `$USERADMIN_UID` - The UID of the Unix user.
  * `$USERADMIN_REAL` - The real name of the Unix user.
  * `$USERADMIN_SHELL` - The shell of the Unix user.
  * `$USERADMIN_HOME` - The home directory of the Unix user.
  * `$USERADMIN_PASS` - The plain-text password of the Unix user, if one was entered by the admin.
  * `$USERADMIN_GROUP` - The name of the Unix group that was just created, modified or deleted.

---

> #### When installing the Webmin _rpm_ package, I get the error message _Unable to identify operating system_?
This happens if Webmin cannot identify your OS by looking at your `/etc/issue` file, possibly because it has been changed from the default contents. The best solution is to install the `.tar.gz` version of Webmin, which asks for the OS name and version manually.

---

> #### How can I create a Webmin user who can only configure one Apache virtual server or DNS domain?
In the **Webmin Users** module, create a new user and give him access to only the Apache Webserver module. After saving, click on Apache webserver next to the user's name in the list of Webmin users and use the form that appears to deny him access to everything except one selected virtual server.
Many other modules can also be configured in a similar way to restrict the access of a user to only certain DNS domains, Unix users or mail aliases.

---

> #### Is there a version of Webmin for Windows?
No, Webmin currently is not supported on Windows.

---

> #### How can I change Webmin's list of allowed IP addresses from the shell?
The file you need to modify is `/etc/webmin/miniserv.conf` , in particular the `allow=` or `deny=` lines. If the `allow=` line exists, it contains a list of all addresses and networks that are allowed to connect to Webmin. Similarly, the `deny=` line contains addresses that are not allowed to connect. After modifying this file, you need to run `/etc/webmin/restart` for the changes to take effect. Naturally, the file can only be edited by the _root_ user.

---

> #### After logging into Webmin, I get the error message _You do not have access to any Webmin modules_?
We are not sure how this error happens, but if it does you can follow these steps to fix it:
* Login to your server via telnet or at the console as _root_.
* Edit the file `/etc/webmin/webmin.acl` and make sure the line starting with `root:` or `admin:` (depending on which you use to login to Webmin) exists and looks like:
   ```text
   root: acl
   ```
* Login to Webmin again, and go into the **Webmin Users** module, which will be the only one you have access to.
* Click on your username in the list, grant yourself access to all the modules, and click **Save**.

---

> #### Can Webmin be run from `inetd`?
    
Yes, with some small changes to the config files. The steps you need to follow are:

*   Stop Webmin with the command `/etc/webmin/stop`
    
*   Add the line `inetd=1` to `/etc/webmin/miniserv.conf`
    
*   Remove the line `session=1` from `/etc/webmin/miniserv.conf`
    
*   Edit `/etc/services` and add a line like:  
    ```text
    webmin 10000/tcp  
    ```
    
*   Edit `/etc/inetd.conf` and add a line like:
    ```text
    webmin stream tcp nowait root /usr/libexec/webmin/miniserv.pl miniserv.pl /etc/webmin/miniserv.conf
    ```
    If you have installed Webmin somewhere else, you will have to change the `/usr/libexec/webmin` part of the path above.
    
*   Restart `inetd` to make the changes take effect. You should now be able to access Webmin on port 10000 as normal.  
    
*   Using the **Bootup and Shutdown** module, make sure that `inetd` is configured to start at boot time, and that webmin is not.  
    

If you are using `xinetd` instead of `inetd`, follow these steps instead:

*   Stop Webmin with the command `/etc/webmin/stop`
    
*   Add the line `inetd=1` to `/etc/webmin/miniserv.conf`
    
*   Remove the line `session=1` from `/etc/webmin/miniserv.conf`
    
*   Edit `/etc/xinetd.conf` and add a section like :
    ```shell
    service webmin
    {
        user = root
        env = LANG=
        port = 10000
        socket_type = stream
        protocol = tcp
        wait = no
        disable = no
        type = UNLISTED
        server = /usr/libexec/webmin/miniserv.pl
        server_args = /etc/webmin/miniserv.conf
    }
    ```

    If you have installed Webmin somewhere else, you will have to change the `/usr/libexec/webmin` part of the path above.
    
*   Restart `xinetd` to make the changes take effect. You should now be able to access Webmin on port 10000 as normal.  
    
*   Using the **Bootup and Shutdown** module make sure that `xinetd` is configured to start at boot time, and that webmin is not.  
    

To run Usermin from `inetd` or `xinetd`, follow the exact same steps but replace /etc/webmin with /etc/usermin and change the port to 2.010.


---

> #### How can I make a Webmin user always use the same password as Unix user?
This can be done by following these steps :
* In the **Perl Modules** module of Webmin, install `Authen::PAM`.
* In the **PAM Authentication** module, add a new PAM service called Webmin that uses Unix authentication.
* In the **Webmin Users** module, click on the user that you want to symchronize with Unix and set his **Password** option to **Unix Authentication**.
If PAM is not used on your operating system, the first two steps can be skipped. Webmin will instead read the `/etc/passwd` or `/etc/shadow` file directly to authenticate users who are using the **Unix Authentication** password mode.

---

> #### How can I uninstall Webmin?
Just run the command `/etc/webmin/uninstall.sh`. If you have installed the _rpm_ package of Webmin, you can also use `rpm -e webmin`, or `dpkg -r webmin` if you have installed the _deb_ package, or if you have installed the Solaris package you can use `pkgrm WSwebmin` command.

---

> #### How can I allow any Unix user to login to Webmin?
Follow these steps:
* In the **Perl Modules** module of Webmin, install `Authen::PAM`.
* In the **PAM Authentication** module, add a new PAM service called webmin that uses **Unix authentication**.
* In the **Webmin Users** module, create a new user called something like _unixer_, with access to the modules that you want all your Unix users to have access to.
* In each of the modules _unixer_ has access to, change the module access control to give your users rights only to their own accounts. For example, in the **Change Passwords** module you should select **Only this user** for the **Users whose passwords can be changed** so that Unix users logging in can only change their own passwords.
* Click on **Configure Unix user authentication** below the list of Webmin users and choose **Allow any Unix user to login with permissions of user** _unixer_.
* Any Unix user should now be able to login to Webmin on your system.
Again, if your system does not use PAM the first two steps can be skipped, and Webmin will read `/etc/passwd` or `/etc/shadow` file directly to authenticate users.
Another alternative to doing all this is to install Usermin, which allows all Unix users to login and access only settings belonging to them, using a similar interface to Webmin.

---

> #### How do I upgrade the Solaris package version of Webmin?
By default, Solaris doesn't allow packages to be upgraded. However, you can change this by editing the file `/var/sadm/install/admin/default` and changing the `instance=` line to `instance=overwrite`. An upgrade can then be performed by simply installing the new Webmin `.pkg` file.

---

> #### In Usermin's _Read Mail_ module, how can I set users' _From_ addresses when my server hosts multiple virtual domains?
By default, when a user composed email the _From_ field contains _username_@_systemhostname_. This can be changed by following these steps:
* Login to Webmin on the same server, and enter the **Usermin Configuration** module.
* Click on **Usermin Module Configuration**.
* Click on **Read Mail**.
* In the **Default hostname for From: addresses** field, enter the domain or hostname that you want to appear after the `@` in users' _From_ addresses.
* If you want to stop users from changing their _From_ address (to prevent mail forging), set the **Allow editing of From: address** option to **No**.
* If you have multiple virtual domains and want different users to have different domains in their _From_ addresses, you will need to set the **From: address mapping file** to the name of a file that maps real email addresses to virtual domain email addresses. This must be a text file, with each line containing :
```text
username     fromaddress
```
The _username_ part of each line must be the user's Usermin login, and the _fromaddress_ is the new _From_ address to assign to that user. The _username_ can also be the user's full email address as it currently appears, such as _joe@yourserver.com_.

---

> #### In Usermin's _MySQL Database_ module, how can I restrict the databases that each user can see and use?
By default the module will list all of the databases on your system on the main page, even if some are not actually usable by the logged-in user. To change this, follow these steps:
* Login to Webmin on the same server, and enter the **Usermin Configuration** module.
* Click on **Usermin Module Configuration**.
* Click on **MySQL Database** in the list.
* In the **Database access control list** field, remove the existing `*: *` line and enter one line per user, containing the username, a colon and list of databases he is allowed to use. For example, you could enter:
```text
jamie: database1
joe: database2 database3
ilia: *
```
A `*` in the database column means all databases, while a `*` in the username column means any user not listed so far.
* Hit the **Save** button to activate the restrictions.

---

> #### Why do reports for different logs generated in the _Webalizer_ module come out the same?
This often happens on Red Hat Linux systems (and derivatives) due to a bug (in our opinion) in the default Webalizer configuration. To fix it, do the following :
* Edit the file `/etc/webalizer.conf`.
* Change the line starting with `HistoryName` to `HistoryName webalizer.hist`.
* Change the line starting with `IncrementalName` to `IncrementalName webalizer.current`.
* Make the same change to any `*.conf` files in `/etc/webmin/webalizer`.
* Re-generate all reports.

---

> #### Why do downloads made from within Webmin fail, when other programs like `wget` work fine?
If you have a firewall that transparently proxies outgoing HTTP requests (such as one by Sonicwall), this may cause requests made by Webmin to be timed out. Without going into the underlying protocol details, my investigation has shown that Sonicwall is making incorrect assumptions about the number of IP packets an HTTP request will be in, and is thus *broken*.
The work-around is to disable the **Enforce Host Tag Search** option in the firewall, which turns off this broken feature.

---

> #### What ports does Webmin RPC use
Webmin has two RPC modes:
- Slow mode, that only uses the same HTTP port the webserver listens on (typically 10000).
- Fast mode which uses ports 10000 on up. The upper bound depends on the number of concurrent RPC operations, but opening the range 10000 to 10010 should be enough when configuring the firewall between two Webmin servers.

---

> #### What does the error _pam_ck_connector(webmin:session): cannot determine display-device_ mean?
If you see this error in `/var/log/auth.log`, edit the file `/etc/pam.d/webmin` and change the line `@include common-session` to `@include common-session-noninteractive` . Then run `/etc/webmin/restart` .
