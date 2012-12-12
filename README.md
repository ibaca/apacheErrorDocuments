# Customizing Apache Error Documents

## Debian
* This is on Debian based systems. Written from Ubuntu 12.04 LTS

Apache's default error pages are served from: 

```
/usr/share/apache2/error
```

To customize these pages, we'll make a copy of them somewhere in `/var/www` that we'll edit and tell Apache to use our new ones instead. I usually only keep my default stuff in `/var/www/`, so I'll put them directly into `/var/www`.

```
root@magikarp:/var/www# ll
total 20
drwxr-xr-x  4 root root 4096 Dec 12 16:06 ./
drwxr-xr-x 13 root root 4096 Nov 18 16:55 ../
drwxr-xr-x  6 root root 4096 Dec 12 11:25 assets/
drwxr-xr-x  3 root root 4096 Dec 12 15:49 error/ <- Our copy
-rw-r--r--  1 root root 2300 Dec 12 16:00 index.html
root@magikarp:/var/www# 
```
If you `ls` our new `error` directory, you'll see a bunch of `.html.var` files for all of our different errors. To make these pages prettier, though, we're mostly interested in `error/include`.

```
root@magikarp:/var/www/error/include# ll
total 16
drwxr-xr-x 2 root root 4096 Dec 12 16:01 ./
drwxr-xr-x 3 root root 4096 Dec 12 15:49 ../
-rw-r--r-- 1 root root  422 Dec 12 15:46 bottom.html
-rw-r--r-- 1 root root    0 Dec 12 15:44 spacer.html
-rw-r--r-- 1 root root 2264 Dec 12 16:01 top.html
root@magikarp:/var/www/error/include#
```

`top.html` is spit out before the error message. So this is where we want to put all our styles and whatnot. `bottom.html` is, as you guessed it, spit out after the error message. 

### Required modules
You need to make sure these three modules are enabled. 

 - mod_negotiation
 - mod_include
 - mod_alias

```
root@magikarp:/etc/apache2/conf.d# a2enmod negotiation include alias
Module negotiation already enabled
Module include already enabled
Module alias already enabled
root@magikarp:/etc/apache2/conf.d# 
```

### Config File
To add our changes to the apache config, there should be a template already at `/etc/apache2/conf.d/localized-error-pages`. Do a quick `cp localized-error-pages localized-error-pages.bak` to make a backup, and uncomment it to look like this:

```
root@magikarp:/etc/apache2/conf.d# cat localized-error-pages
<IfModule mod_negotiation.c>
 <IfModule mod_include.c>
  <IfModule mod_alias.c>

    Alias /error/ "/var/www/error/"

    <Directory "/var/www/error">
        AllowOverride None
        Options IncludesNoExec
        AddOutputFilter Includes html
        AddHandler type-map var
        Order allow,deny
        Allow from all
        LanguagePriority en cs de es fr it nl sv pt-br ro
        ForceLanguagePriority Prefer Fallback
    </Directory>

    ErrorDocument 400 /error/HTTP_BAD_REQUEST.html.var
    ErrorDocument 401 /error/HTTP_UNAUTHORIZED.html.var
    ErrorDocument 403 /error/HTTP_FORBIDDEN.html.var
    ErrorDocument 404 /error/HTTP_NOT_FOUND.html.var
    ErrorDocument 405 /error/HTTP_METHOD_NOT_ALLOWED.html.var
    ErrorDocument 408 /error/HTTP_REQUEST_TIME_OUT.html.var
    ErrorDocument 410 /error/HTTP_GONE.html.var
    ErrorDocument 411 /error/HTTP_LENGTH_REQUIRED.html.var
    ErrorDocument 412 /error/HTTP_PRECONDITION_FAILED.html.var
    ErrorDocument 413 /error/HTTP_REQUEST_ENTITY_TOO_LARGE.html.var
    ErrorDocument 414 /error/HTTP_REQUEST_URI_TOO_LARGE.html.var
    ErrorDocument 415 /error/HTTP_UNSUPPORTED_MEDIA_TYPE.html.var
    ErrorDocument 500 /error/HTTP_INTERNAL_SERVER_ERROR.html.var
    ErrorDocument 501 /error/HTTP_NOT_IMPLEMENTED.html.var
    ErrorDocument 502 /error/HTTP_BAD_GATEWAY.html.var
    ErrorDocument 503 /error/HTTP_SERVICE_UNAVAILABLE.html.var
    ErrorDocument 506 /error/HTTP_VARIANT_ALSO_VARIES.html.var
  </IfModule>
 </IfModule>
</IfModule>
root@magikarp:/etc/apache2/conf.d# 
```
### Restart Apache
Restart apache and your new error documents should be getting served!
