# Strapi on OpenLiteSpeed

Why do it simple when you can do it weird?

## Why would you do that?

When it comes to web servers, it's usually a question of nginx or apache. Recently, it might not even be a question anymore considering tools like [netlify](https://www.netlify.com/ "Netlify’s website") or [vercel](https://vercel.com/ "Vercel’ website") handle almost all backend configuration on their own.

However, when installing Strapi the regular way, we need to choose. I chose [OpenLiteSpeed.](https://openlitespeed.org "OpenLiteSpeed’s website")

### Benefits of OpenLiteSpeed

- it has a graphical interface
- it has a good [documentation](https://openlitespeed.org/support/ "OpenLiteSpeed’s documentation")
- http/3 (quic) support out of the box
- and more [features](https://openlitespeed.org/#features "list of OLS features")

## Installing OpenLiteSpeed

OpenLiteSpeed can be installed on a server (ubuntu/debian/centOS) through [official repositories](https://openlitespeed.org/kb/install-ols-from-litespeed-repositories/) or using [a pre-configured image](https://do.co/2LUwBVp "OpenLiteSpeed image on Digital Ocean") like Digital Ocean offers.

I tried both and everything works fine.

## Virtual host configuration

### Basic tab

- login into your admin panel, usually accessible on port 7080 (your.ip.address:7080)
- add a new virtual host following [the official guide](https://openlitespeed.org/kb/setting-up-name-based-virtual-hosting-on-openlitespeed/ "New virtual host guide (new window)")
- the _Virtual Host Name_ can be anything like "strapi"
- the _Virtual host root_ can be the folder where strapi lives
- for the config file, you can use this string: `$SERVER_ROOT/conf/vhosts/$VH_NAME/$VH_NAME.conf`  
  It uses server variables to put the conf inside `/usr/local/lsws/conf/vhosts/strapi/strapi.conf`
  When you save, OLS will ask you if it should create the .conf file for you, say yes.

![](https://user-images.githubusercontent.com/20305403/140758038-2e47f961-6a42-4dc3-90df-c6b850779fe8.png)

### General tab

- the _Document Root_ does not matter, I set it to the `public` strapi folder
- set the _Domain name_ to your strapi domain

![](https://user-images.githubusercontent.com/20305403/140758724-46c7cb70-1f66-49ae-b3a0-a288f6338233.png)

### External App tab

- add a new 'Web Server' in the _external app_ tab
- give it an obvious name
- set the address to strapi's default `0.0.0.0:1337`
- you can define environment variables in _Environment_ instead of a .env file if you wish

![](https://user-images.githubusercontent.com/20305403/140758868-75a12987-f449-4a88-9eac-7009fa5b90d8.png)

### Context tab

- add a new 'Proxy' context
- use the root `/` for the 'URI' field
- select your previously created web server form the list
- you can set headers if you want in the _Header Operations_ field
- allowed access to everyone with `*` inside _Access Allowed_

![](https://user-images.githubusercontent.com/20305403/140759885-ac1cb611-e0da-4779-9113-a16bccc40edf.png)

### SSL tab

Here you can define a TLS certificate to be used by OLS for your domain.

![](https://user-images.githubusercontent.com/20305403/140760305-6bb4887c-0972-420e-97be-f4d6419bf60b.png)



## Listener configuration

On the listeners window [bind your virtual host to both http and https (ssl) listeners.](https://openlitespeed.org/kb/setting-up-name-based-virtual-hosting-on-openlitespeed/#Create_and_Assign_Listeners "Binding virtual host to listener guide (new window)")

It’s a pretty straightforward process. Or so I thought.

In order to serve both ipv4 and ipv6 [LiteSpeed recommends to use separate listeners.](https://www.litespeedtech.com/docs/webserver/config/listener-general "Listener LiteSpeed documentation (new window)") However, I ran into some problems trying to configure separate IPv4/IPv6 listeners so here's how to do it with one.

### General tab

- on the _Address Settings_, choose `[ANY] IPv6` for the _IP Address_ field.
- the _Port_ field depends on http/https (80/443) but **be sure to do both listeners.**

On the _Virtual Host Mappings_, add a new host and choose your Strapi virtual host from the drop-down. In order to listen to ipv4 and ipv6, we need to specify both IP adresses and use a particular syntax for the ipv4 by prepending it with `::FFFF:` as such: ` strapi.yourdomain.tld, ::FFFF:IPV4, IPV6`

Do the same for the secure listener on port 443

![](https://user-images.githubusercontent.com/20305403/140761377-6d5c43ab-189c-4a4a-91ae-c151e959af7d.png)

## Deploying Strapi

Now that the server is ready, we can deploy strapi.
I followed the [official guide](https://strapi.io/documentation/developer-docs/latest/setup-deployment-guides/deployment.html#deployment "Strapi deployment guides"). It's basically `NODE_ENV=production yarn build` and `NODE_ENV=production yarn start`

It's possible (and recommended) to automate this with pm2.

## We’re done
Strapi should be accessible through its domain.  
Don’t forget to perform a _graceful restart_ of OpenLiteSpeed and close port 7080 when you’re done with the admin panel.
