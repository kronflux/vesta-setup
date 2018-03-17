## Nginx HTTP/2 with GeoIP, ngx_cache_purge and Google's ngx_pagespeed support

Installing a new version of Nginx from source will mean that you can no longer receive updates for Nginx from apt-get.

1. Install packages used for compiling
```
apt-get install build-essential dpkg-dev geoip-database libgd2-xpm-dev libgeoip-dev libgeoip1 libpcre3 libpcre3-dev libpcrecpp0v5 libperl-dev logrotate unzip zip zlib1g-dev
```
2. Grab latest GeoIP data
```
cd /usr/share/GeoIP  
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz  
wget http://geolite.maxmind.com/download/geoip/database/GeoIPv6.dat.gz  
mv GeoIP.dat GeoIP.dat.bak  
mv GeoIPv6.dat GeoIPv6.dat.bak
gunzip GeoIP.dat.gz  
gunzip GeoIPv6.dat.gz
```
3. Go to the source directory
```
cd /usr/local/src/
```
4. Get newer version of OpenSSL. (This is for compling only, do not replace OpenSSL on your OS.)
```
OPENSSL_VERSION=1.1.1-pre2
wget -O openssl-${OPENSSL_VERSION}.tar.gz https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz
tar -xzvf openssl-${OPENSSL_VERSION}.tar.gz
```
5. Get ngx_cache_purge module
```
CACHEPURGE_VERSION=2.4.2
wget -O ngx_cache_purge-${CACHEPURGE_VERSION}.tar.gz https://github.com/nginx-modules/ngx_cache_purge/archive/${CACHEPURGE_VERSION}.tar.gz
tar -xzvf ngx_cache_purge-${CACHEPURGE_VERSION}.tar.gz
```
6. Get ngx_pagespeed module and psol libraries
```
PAGESPEED_VERSION=v1.13.35.2-stable
PSOL_VERSION=1.13.35.2
wget -O ngx_pagespeed-${PAGESPEED_VERSION}.zip https://github.com/apache/incubator-pagespeed-ngx/archive/v${PAGESPEED_VERSION}.zip
unzip ngx_pagespeed-${PAGESPEED_VERSION}.zip
mv incubator-pagespeed-ngx-${PAGESPEED_VERSION} ngx_pagespeed-${PAGESPEED_VERSION}
cd ngx_pagespeed-${PAGESPEED_VERSION}/
wget -O ngx_pagespeed_psol-${PSOL_VERSION}-x64.tar.gz https://dl.google.com/dl/page-speed/psol/${PSOL_VERSION}-x64.tar.gz
tar -xzvf ngx_pagespeed_psol-${PSOL_VERSION}-x64.tar.gz
rm ngx_pagespeed_psol-${PSOL_VERSION}-x64.tar.gz
```
7. Get newer version of Nginx
```
cd /usr/local/src  
NGINX_VERSION=1.13.9
wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz  
tar -xvzf nginx-${NGINX_VERSION}.tar.gz  
cd nginx-${NGINX_VERSION}/
```
8. Stop the existing Nginx service
```
service nginx stop
```
9. Set config with new params
```
./configure --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-file-aio --with-threads --with-ipv6 --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_geoip_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-cc-opt='-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -Wl,--as-needed' --with-openssl=/usr/local/src/openssl-${OPENSSL_VERSION} --add-module=/usr/local/src/ngx_cache_purge-${CACHEPURGE_VERSION} --add-module=/usr/local/src/ngx_pagespeed-${PAGESPEED_VERSION}
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

Now we need to edit the VestaCP templates to use Pagespeed and HTTP2.
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
pagespeed On;
pagespeed RewriteLevel CoreFilters;

# HTTPS Support
pagespeed FetchHttps enable,allow_self_signed;

pagespeed EnableFilters lazyload_images,collapse_whitespace,insert_dns_prefetch,dedup_inlined_images,defer_javascript,pedantic,trim_urls,sprite_images,extend_cache_pdfs,remove_comments,resize_mobile_images,inline_preview_images,insert_image_dimensions,convert_to_webp_lossless,local_storage_cache,inline_google_font_css,prioritize_critical_css,rewrite_style_attributes,move_css_to_head,move_css_above_scripts,outline_javascript,outline_css,combine_heads;

pagespeed FileCachePath /var/ngx_pagespeed_cache;

location ~ "\.pagespeed\.([a-z]\.)?[a-z]{2}\.[^.]{10}\.[^.]+" { add_header "" ""; }
location ~ "^/pagespeed_static/" { }
location ~ "^/ngx_pagespeed_beacon$" { }
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
