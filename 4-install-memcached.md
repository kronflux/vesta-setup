## Install Memcached (for PHP7-FPM)

1. Install the Memcached packages
```
apt-get install php7.0-memcached memcached
```
2. Test Memcached:
```
php -a
```
Paste the following:
```
$m = new Memcached();  
$m->addServer('127.0.0.1', 11211);  
$m->set('foo', 100);  
echo $m->get('foo') . "\n";  
```
If 100 returned, all is good.

You can also paste the above in to a .php on the server to test it via web browser.

### WordPress with Memcached

Download the [Memcached plugin](https://wordpress.org/plugins/memcached/). Extract the object-cache.php file. Upload it to wp-content of your WP Install.
