## Nginx HTTP/2

Installing a new version of Nginx from source will mean that you can no longer receive updates for Nginx from apt-get.

1. Install package used for complie

apt-get install dpkg-dev libpcrecpp0 libgd2-xpm-dev libgeoip-dev libperl-dev libpcre3 libpcre3-dev

2. Go to the source directory

cd /usr/local/src/

2. Get newer version of OpenSSL.

This is for compling only, do not replace OpenSSL on your OS.

wget https://www.openssl.org/source/openssl-1.0.2h.tar.gz
tar -xzvf openssl-1.0.2h.tar.gz

3. Get newer version of Nginx.

NGINX_VERSION=1.11.6
wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
tar -xvzf nginx-${NGINX_VERSION}.tar.gz
cd nginx-${NGINX_VERSION}/

3. Stop the existing Nginx service

service nginx stop

3. Set config with new params.

./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-file-aio --with-threads --with-ipv6 --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed' --with-openssl=/usr/local/src/openssl-1.0.2h

4. Make / Install

make
make install

5. Restart nginx

service nginx restart

6. Check you have the new version

nginx -v

7. Test it out.

Create a new domain in VestaCP Web GUI with SSL Enabled.
If testing locally, then generate a CSR and paste in the SSL Key.

If on a real server use the new Let's Encrpt VestaCP GUI to install an SSL Cert for your domain.

Edit the snginx.conf file for the domain

e.g. /home/admin/conf/web/snginx.conf

add 'ssl http2' to the listen line of nginx config file for your domain.

server {
	listen 192.168.3.151:443 ssl http2;
	...
}

Note: Might be better to update the main templates stored by VestaCP templates and rebuild from the Web GUI.

service nginx restart

Install the Chrome HTTP/2 and SPDY indicator.

https://chrome.google.com/webstore/detail/http2-and-spdy-indicator/mpbpobfflnpcgagjijhmgnchggcjblin

Access your new domain on SSL in Chrome, e.g. https://testbed.dev.

The lightening blot indicator from the Chrome extension should be Blue. If you click on it you'll see the domain in chrome://net-internals/#http2

Alternatively open update the Network tab of Chrome Dev Tools and show the Protocol Column. Refresh the page. Protocol should show as H2 for requests made on the server.

Google fonts and other external resource may show SPDY or HTTP/1..
