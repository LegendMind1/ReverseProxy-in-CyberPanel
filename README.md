# ReverseProxy-in-CyberPanel

 ## vHost Conf Setting for No Proxy in Cyber Panel
 ```console
 docRoot                   $VH_ROOT/public_html
vhDomain                  $VH_NAME
vhAliases                 www.$VH_NAME
adminEmails               spaceisnotnull@gmail.com
enableGzip                1
enableIpGeo               1

index  {
  useServer               0
  indexFiles              index.php, index.html
}

errorlog $VH_ROOT/logs/$VH_NAME.error_log {
  useServer               0
  logLevel                WARN
  rollingSize             10M
}

accesslog $VH_ROOT/logs/$VH_NAME.access_log {
  useServer               0
  logFormat               "%h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i""
  logHeaders              5
  rollingSize             10M
  keepDays                10  
  compressArchive         1
}

scripthandler  {
  add                     lsapi:legen5300 php
}

extprocessor legen5300 {
  type                    lsapi
  address                 UDS://tmp/lshttpd/legen5300.sock
  maxConns                10
  env                     LSAPI_CHILDREN=10
  initTimeout             600
  retryTimeout            0
  persistConn             1
  pcKeepAliveTimeout      1
  respBuffer              0
  autoStart               1
  path                    /usr/local/lsws/lsphp80/bin/lsphp
  extUser                 legen5300
  extGroup                legen5300
  memSoftLimit            2047M
  memHardLimit            2047M
  procSoftLimit           400
  procHardLimit           500
}

phpIniOverride  {

}

module cache {
 storagePath /usr/local/lsws/cachedata/$VH_NAME
}

rewrite  {
 enable                  1
  autoLoadHtaccess        1
}

context /.well-known/acme-challenge {
  location                /usr/local/lsws/Example/html/.well-known/acme-challenge
  allowBrowse             1

  rewrite  {
     enable                  0
  }
  addDefaultCharset       off

  phpIniOverride  {

  }
}
```

 ## vHost Conf Setting for Reverse Proxy in Cyber Panel
 ```console
docRoot                   $VH_ROOT/public_html
vhDomain                  $VH_NAME
vhAliases                 www.$VH_NAME
adminEmails               spaceisnotnull@gmail.com
enableGzip                1
enableIpGeo               1

index  {
  useServer               0
  indexFiles              index.php, index.html
}

errorlog $VH_ROOT/logs/$VH_NAME.error_log {
  useServer               0
  logLevel                WARN
  rollingSize             10M
}

accesslog $VH_ROOT/logs/$VH_NAME.access_log {
  useServer               0
  logFormat               "%h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i""
  logHeaders              5
  rollingSize             10M
  keepDays                10  
  compressArchive         1
}

scripthandler  {
  add                     proxy:legen5300 php
}

extprocessor legen5300 {
  type proxy
address 127.0.0.1:3001
maxConns 2000
initTimeout 20
retryTimeout 0
respBuffer 0
}

phpIniOverride  {

}

module cache {
 storagePath /usr/local/lsws/cachedata/$VH_NAME
}

rewrite  {
RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
RewriteCond %{HTTP:CONNECTION} Upgrade$ [NC]
RewriteRule /(.*) ws://127.0.0.1:3001/$1 [P]
 enable                  1
  autoLoadHtaccess        1
}

context /.well-known/acme-challenge {
  location                /usr/local/lsws/Example/html/.well-known/acme-challenge
  allowBrowse             1

  rewrite  {
     enable                  0
  }
  addDefaultCharset       off

  phpIniOverride  {

  }
}

context / {
type proxy
handler legen5300
addDefaultCharset off
}
```
