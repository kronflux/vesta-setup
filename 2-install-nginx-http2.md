## Nginx HTTP/2 with GeoIP, ngx_cache_purge and quiche

Installing a new version of Nginx from source will mean that you can no longer receive updates for Nginx from apt-get.

1. Install packages used for compiling
```
apt-get install build-essential dpkg-dev geoip-database libgd2-xpm-dev libgeoip-dev libgeoip1 libpcre3 libpcre3-dev libpcrecpp0v5 libperl-dev logrotate unzip zip zlib1g-dev
```
2. Grab latest GeoIP data (GeoLite2 Legacy Converted)
```
cd /usr/share/GeoIP  
wget https://dl.miyuru.lk/geoip/maxmind/country/maxmind4.dat.gz
wget https://dl.miyuru.lk/geoip/maxmind/country/maxmind6.dat.gz
mv GeoIP.dat GeoIP.dat.bak
mv GeoIPv6.dat GeoIPv6.dat.bak
gunzip maxmind4.dat.gz
gunzip maxmind6.dat.gz
mv maxmind4.dat GeoIP.dat
mv maxmind6.dat GeoIPv6.dat
```
3. Go to the source directory
```
cd /usr/local/src/
```
4. Get newer version of OpenSSL. (This is for compling only, do not replace OpenSSL on your OS.)
```
OPENSSL_VERSION=1.1.1g
wget -O openssl-${OPENSSL_VERSION}.tar.gz https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz
tar -xzvf openssl-${OPENSSL_VERSION}.tar.gz
rm openssl-${OPENSSL_VERSION}.tar.gz
```
5. Get ngx_cache_purge module
```
CACHEPURGE_VERSION=2.5.1
wget -O ngx_cache_purge-${CACHEPURGE_VERSION}.tar.gz https://github.com/nginx-modules/ngx_cache_purge/archive/${CACHEPURGE_VERSION}.tar.gz
tar -xzvf ngx_cache_purge-${CACHEPURGE_VERSION}.tar.gz
rm ngx_cache_purge-${CACHEPURGE_VERSION}.tar.gz
```
7. Get quiche module
```
QUICHE_VERSION=0.5.1
wget -O quiche-${QUICHE_VERSION}.tar.gz https://github.com/cloudflare/quiche/archive/${QUICHE_VERSION}.tar.gz
tar -xzvf quiche-${QUICHE_VERSION}.tar.gz
rm quiche-${QUICHE_VERSION}.tar.gz
```
6. Get newer version of Nginx
```  
NGINX_VERSION=1.19.1
wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
tar -xzvf nginx-${NGINX_VERSION}.tar.gz
rm nginx-${NGINX_VERSION}.tar.gz
cd nginx-${NGINX_VERSION}/
```
7. Stop the existing Nginx service
```
service nginx stop
```
8. Mark the Nginx package on hold in Apt
```
apt-mark hold nginx nginx-full nginx-common
```
9. Set config with new params
```
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-file-aio --with-threads --with-ipv6 --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_geoip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-http_v3_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed' --with-openssl=/usr/local/src/openssl-${OPENSSL_VERSION} --add-module=/usr/local/src/ngx_cache_purge-${CACHEPURGE_VERSION}
```
10. Make / Install
```
make
make install
```
11. Restart nginx
```
service nginx restart
```
12. Check you have the new version
```
nginx -v
```
13. Test it out

Create a new domain in VestaCP Web GUI with SSL Enabled.
If testing locally, then generate a CSR and paste in the SSL Key.

If on a real server use the new Let's Encrpt VestaCP GUI to install an SSL Cert for your domain.

Now we need to edit the VestaCP templates to use HTTP2.
In the following files:
```
/usr/local/vesta/data/templates/web/nginx/default.tpl
/usr/local/vesta/data/templates/web/nginx/hosting.tpl
/usr/local/vesta/data/templates/web/nginx/caching.tpl
/usr/local/vesta/data/templates/web/nginx/default.stpl
/usr/local/vesta/data/templates/web/nginx/hosting.stpl
/usr/local/vesta/data/templates/web/nginx/caching.stpl
/usr/local/vesta/data/templates/web/nginx/php-fpm/default.tpl
/usr/local/vesta/data/templates/web/nginx/php-fpm/default.stpl
```
add:
```
# HTTPS Support

```
before the include line which looks something like this:
```
include %home%/%user%/conf/web/nginx.%domain%.conf*;
```
In the files ending in .stpl change the listen line from:
```
listen      %ip%:%proxy_ssl_port%;
```
to:
```
listen      %ip%:%proxy_ssl_port% ssl http2;
```
Now, restart nginx.
```
service nginx restart
```
Install the Chrome HTTP/2 and SPDY indicator.

https://chrome.google.com/webstore/detail/http2-and-spdy-indicator/mpbpobfflnpcgagjijhmgnchggcjblin

Access your new domain on SSL in Chrome, e.g. https://testbed.dev.

The lightening blot indicator from the Chrome extension should be Blue. If you click on it you'll see the domain in chrome://net-internals/#http2

Alternatively open update the Network tab of Chrome Dev Tools and show the Protocol Column. Refresh the page. Protocol should show as H2 for requests made on the server.

Google fonts and other external resource may show SPDY or HTTP/1..
