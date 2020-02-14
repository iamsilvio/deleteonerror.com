---
title: "50kb is enough!"
subtitle: "About nginx caching and static content."
date: 2020-02-13T21:51:10+01:00
draft: false
tags: ["nginx", "cacheing", "gzip", "mimetype", "StaticContent","hugo"]
categories: [webserver, itstuff]
---

Now that I have found a new blog software it is time to get the web server up and running.
First we teach nginx a little bit about compression with gzip.

## Step 1: gzip everything

By default gzip is already enabled in `/etc/nginx/nginx.conf` but the important part of the configuration is commented out.
<!--more-->
``` nginx

gzip on;
gzip_static on;
gzip_disable "MSIE [1-6]\.(?!.*SV1)";

gzip_vary on;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;

gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript application/javascript text/x-js font/ttf font/opentype  application/vnd.ms-fontobject image/svg+xml;

```

Next, we need to introduce nginx to some ttf and otf mime types, which are in `/etc/nginx/mime.types`

``` nginx
...

    font/ttf                              ttf;
    font/opentype                         otf;
...

```

## Step 2: Cache static content

The configuration is quite self-explanatory, depending on the file extension we add
different cache times and deactivate the access log.

``` nginx

    location ~* \.(?:ico)$ {
      expires 30d;
      add_header Cache-Control public;
      access_log off;
    }

    location ~* \.(?:css|js) {
      expires 7d;
      add_header Cache-Control public;
      access_log off;
    }

    location ~* \.(?:eot|ttf|svg)$ {
      expires 365d;
      add_header Vary Accept-Encoding;
      add_header Cache-Control public;
      access_log off;
    }

    location ~* \.(?:woff)$ {
      expires 365d;
      add_header Cache-Control public;
      access_log off;
    }

    location ~* \.(?:gif|png|jpe?g)$ {
      expires 30d;
      add_header Cache-Control public;
      access_log off;
    }

```

surprise, it's that simple!

## Update

Did i mention that it is not a good idea to cache xml and json?

So we have one addition to our locations

``` nginx

    location ~* \.(?:xml|json) {
      add_header Last-Modified $date_gmt;
      add_header Cache-Control 'no-store, no-cache, must-revalidate, proxy-revalidate,max-age=0';
      if_modified_since off;
      expires off;
      etag off;
    }

```
