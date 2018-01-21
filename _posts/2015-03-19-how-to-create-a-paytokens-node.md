---
title: How to Create a Paytokens Node
date: 2015-03-19 00:00:00 Z
categories:
- blog
layout: post
---

Paytokens Servers are currently only supported on 64-Bit Ubuntu 14.04 LTS.  As of this writing, it is recommend to have a system with at least 32 GB of disk space accompanied by 4 GB of RAM and an Intel i5 equivalent or better.  A node targets 1,000 Users.  The install user must be a sudoer or have the appropriate access rights.  Core Paytokens installations are generally to be held by the home directory of the user.  These instructions may change and only serve as a guideline.

To get started, begin with a clean install, enter home directory and update repositories:

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade
{% endhighlight %}

Next, install Litecoin dependencies:

{% highlight bash %}
sudo apt-get install -y git autoconf autogen automake build-essential software-properties-common libboost-all-dev libcurl4-openssl-dev libdb-dev libdb++-dev libgmp3-dev libminiupnpc-dev libmpfr-dev libssl-dev libcurl4-openssl-dev libjansson-dev pax-utils
{% endhighlight %}

Build Litecoin:

{% highlight bash %}
git clone https://github.com/litecoin-project/litecoin.git
cd litecoin/src
make -f makefile.unix USE_UPNP=1
sudo cp ./litecoind /usr/bin
{% endhighlight %}

Litecoind First Run:
Start Litecoin by typing litecoind.  This will provide instructions to setup litecoin.conf in the directory ~/.litecoin.  Please be aware that directory paths should be the full path and exclude ~/ when entering paths later through configuration.  We may omit the full path throughout this writing.  A typical litecoin.conf should resemble the following for Paytokens Nodes:

{% highlight bash %}
rpcuser=litecoinrpc
rpcpassword=PASSWORD
server=1
daemon=1
txindex=1
{% endhighlight %}

Please replace PASSWORD with the password of your choosing.  

Start Litecoind to Rebuild Blockchain Index:

{% highlight bash %}
litecoind -reindex &
{% endhighlight %}

Install Paytokens Dependencies:

{% highlight bash %}
sudo apt-get -y install runit software-properties-common python-software-properties git-core wget cx-freeze python3 python3-setuptools python3-dev python3-pip build-essential python3-sphinx python-virtualenv libsqlite3-dev python3-apsw python3-zmq ntp ssl-cert
{% endhighlight %}

Install Payblock Dependencies:

{% highlight bash %}
sudo apt-get -y install python python-dev python-setuptools python-pip python-sphinx python-zmq libzmq3 libzmq3-dev libxml2-dev libxslt-dev zlib1g-dev libimage-exiftool-perl libevent-dev cython
git clone https://github.com/czarparty/czarpartyd_build.git paytokensd_build
{% endhighlight %}

Install MongoDB Version 2.6.4:

{% highlight bash %}
sudo apt-get -y remove mongodb mongodb-server
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
sudo apt-get update
sudo apt-get -y install mongodb-org=2.6.4 mongodb-org-server=2.6.4 mongodb-org-shell=2.6.4 mongodb-org-mongos=2.6.4 mongodb-org-tools=2.6.4
{% endhighlight %}

Pin this specific version of MongoDB to prevent automatic updates:

{% highlight bash %}
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
{% endhighlight %}

Install Redis:

{% highlight bash %}
sudo apt-get -y install redis-server
{% endhighlight %}

As a note, other versions of Redis may work just fine, that are just not tested within a Paytokens Environment

Install Sqlite Utilities:

{% highlight bash %}
sudo apt-get -y install sqlite sqlite3 libleveldb-dev
{% endhighlight %}

Install Node.js:

{% highlight bash %}
sudo apt-get -y remove nodejs npm gyp
sudo add-apt-repository -y ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get -y install nodejs
{% endhighlight %}

Install Paytokensd:

{% highlight bash %}
cd ~/
git clone https://github.com/paytokens/paytokensd.git
cd paytokensd
sudo pip3 install -r pip-requirements.txt
mkdir data
{% endhighlight %}

Create Paytokensd Settings in "~/paytokensd/settings.py":

{% highlight bash %}
[Default]
data-dir=data
litecoind-rpc-user=litecoinrpc
litecoind-rpc-password=PASSWORD
litecoind-rpc-connect=127.0.0.1
litecoind-rpc-port=8816
rpc-user=paytokensrpc
rpc-password=PASSWORD
rpc-host=127.0.0.1
blockchain-service-name=blockr
blockchain-service-connect=https://ltc.blockr.io
log-file=data/debug.log
{% endhighlight %}

Please match the Litecoin RPC Password to litecoind-rpc-password and Paytokens RPC Password to rpc-password.

Create a shell script to start Paytokens Server in "~/paytokens.sh":

{% highlight bash %}
#!/bin/sh
cd PATH/paytokensd
./paytokensd.py --config-file ./settings.py --data-dir data --blockchain-service-name blockr --blockchain-service-connect https://ltc.blockr.io
{% endhighlight %}

Remember to replace PATH with the correct full path to Paytokensd installation directory.  Make executable: "chmod 750 ~/paytokens.sh"

Install Payblockd:

{% highlight bash %}
cd ~/
git clone https://github.com/paytokens/payblockd.git
cd payblockd
sudo pip2.7 install -r pip-requirements.txt
mkdir data
{% endhighlight %}

Configure Payblockd:
"~/payblockd/settings.py":

{% highlight bash %}
[Default]
data-dir=data
paytokensd-rpc-user=paytokensrpc
paytokensd-rpc-password=PASSWORD
paytokensd-rpc-port=7730
blockchain-service-name=blockr
blockchain-service-connect=https://ltc.blockr.io
log-file=data/debug.log
{% endhighlight %}

Create "payblock.sh":

{% highlight bash %}
#!/bin/sh
cd /home/ceo/payblockd
python2.7 payblockd.py --config-file settings.py --data-dir data --paytokensd-rpc-user paytokensrpc --paytokensd-rpc-password PASSWORD --rpc-host 127.0.0.1 --rpc-port 7800 --blockchain-service-name blockr --blockchain-service-connect https://ltc.blockr.io
{% endhighlight %}

Install Paywallet:

{% highlight bash %}
cd ~/
git clone https://github.com/paytokens/paywallet.git
cd paywallet
sudo npm install -g grunt grunt-cli bower
npm install grunt --save-dev
cd src; bower install; cd ..
npm install
grunt build
{% endhighlight %}

Note that the paywallet repository includes the correct release of livenet.  Do not use the generated build as it will not work.  Instead, use the included build directory 'livenet' for the prebuilt distribution.

Configure Paywallet:
Edit ~/paywallet/livenet/paywallet.conf.json and change api.paytokens.co to localhost or the desired host.

{% highlight bash %}
{
  "servers": [ "https://localhost" ],
  "forceTestnet": false,
  "googleAnalyticsUA": "UA-21065583-20",
  "googleAnalyticsUA-testnet": "UA-21065583-20",
  "rollbarAccessToken": "39d23b5a512f4169c98fc922f0d1b121",
  "disabledFeatures": [ ],
  "restrictedAreas": {
    "pages/betting.html": ["US"],
    "pages/openbets.html": ["US"],
    "pages/matchedbets.html": ["US"],
    "pages/rps.html": ["US"],
    "dividend": ["US"]
  },
  "autoBTCEscrowServer": ""
}
{% endhighlight %}

Install Nginx:

{% highlight bash %}
cd ~/
sudo apt-get -y remove nginx-openresty
sudo apt-get -y remove nginx
sudo apt-get -y install make ruby1.9.1 ruby1.9.1-dev git-core libpcre3-dev libxslt1-dev libgd2-xpm-dev libgeoip-dev unzip zip build-essential libssl-dev
sudo gem install fpm
sudo rm -rf /tmp/openresty /tmp/ngx_openresty-* /tmp/nginx-openresty.tar.gz /tmp/nginx-openresty*.deb
wget -O /tmp/nginx-openresty.tar.gz http://openresty.org/download/ngx_openresty-1.7.2.1.tar.gz
sudo tar -C /tmp -zxvf /tmp/nginx-openresty.tar.gz
sudo cd /tmp/ngx_openresty-1.7.2.1
sudo ./configure --with-luajit --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-log-path=/var/log/nginx/access.log --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --lock-path=/var/lock/nginx.lock --pid-path=/var/run/nginx.pid --with-http_geoip_module --with-http_gzip_static_module --with-http_realip_module --with-http_ssl_module --with-http_sub_module --with-http_xslt_module --with-ipv6 --with-sha1=/usr/include/openssl --with-md5=/usr/include/openssl --with-http_stub_status_module --with-http_secure_link_module --with-http_sub_module
sudo make
sudo cd /tmp/ngx_openresty-1.7.2.1
sudo make install DESTDIR=/tmp/openresty
sudo mkdir -p /tmp/openresty/var/lib/nginx
sudo cp -r ~/paytokensd_build/dist/linux /tmp
sudo install -m 0755 -D /tmp/linux/runit/nginx/run /tmp/openresty/etc/sv/nginx/run
sudo install -m 0755 -D /tmp/linux/nginx/nginx.conf /tmp/openresty/etc/nginx/nginx.conf
sudo install -m 0755 -D /tmp/linux/nginx/counterblock.conf /tmp/openresty/etc/nginx/sites-enabled/counterblock.conf
sudo install -m 0755 -D /tmp/linux/nginx/counterblock_api.inc /tmp/openresty/etc/nginx/sites-enabled/counterblock_api.inc
sudo install -m 0755 -D /tmp/linux/nginx/counterblock_api_cache.inc /tmp/openresty/etc/nginx/sites-enabled/counterblock_api_cache.inc
sudo install -m 0755 -D /tmp/linux/nginx/counterblock_socketio.inc /tmp/openresty/etc/nginx/sites-enabled/counterblock_socketio.inc
sudo install -m 0755 -D /tmp/linux/logrotate/nginx /tmp/openresty/etc/logrotate.d/nginx
sudo cd /tmp
sudo fpm -s dir -t deb -n nginx-openresty -v 1.7.2.1 --iteration 1 -C /tmp/openresty --description "openresty 1.7.2.1" --conflicts nginx --conflicts nginx-common -d libxslt1.1 -d libgeoip1 -d geoip-database -d libpcre3 --config-files /etc/nginx/nginx.conf --config-files /etc/nginx/sites-enabled/counterblock.conf --config-files /etc/nginx/fastcgi.conf.default --config-files /etc/nginx/win-utf --config-files /etc/nginx/fastcgi_params --config-files /etc/nginx/nginx.conf --config-files /etc/nginx/koi-win --config-files /etc/nginx/nginx.conf.default --config-files /etc/nginx/mime.types.default --config-files /etc/nginx/koi-utf --config-files /etc/nginx/uwsgi_params --config-files /etc/nginx/uwsgi_params.default --config-files /etc/nginx/fastcgi_params.default --config-files /etc/nginx/mime.types --config-files /etc/nginx/scgi_params.default --config-files /etc/nginx/scgi_params --config-files /etc/nginx/fastcgi.conf etc usr var
sudo apt-get -y install libxslt1.1 libgeoip1 geoip-database libpcre3
sudo dpkg -i /tmp/nginx-openresty_1.7.2.1-1_amd64.deb
sudo rm -f /etc/nginx/sites-enabled/*.dpkg-old /etc/nginx/sites-enabled/*.dpkg-dist
sudo ln -sf /etc/sv/nginx /etc/service/
{% endhighlight %}

Configure Nginx:

{% highlight bash %}
sudo nano /etc/nginx/sites*/*
{% endhighlight %}

This will bring up an editor to modify Nginx Site Configuration.  Please adjust settings in these files to match desired output.  In particular comment out 'etc var usr'.

You may now start Paytokensd and payblockd then open a browser and begin utilizing the local instance of Paytokens.


