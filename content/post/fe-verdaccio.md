---
title: "Build a private npm registry using verdaccio"
date: "2022-01-07"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---



In this post, we will learn how to create your own npm registry using verdaccio. Why do you bother using a private registry? This is because we do not want to publish our code to the public due to data security and privacy.



<!--more-->



## Install



First, we need to install `verdaccio` and `pm2` globally.



```bash
npm install -g verdaccio
npm install -g pm2
```



`pm2 ` is used to guard our service, the common commands are shown below.



```bash
pm2 start verdaccio

pm2 stop verdaccio

pm2 delete verdaccio
```



To check the service status, we use the following command. `status: online`  shows that everything is ok.



```bash
pm2 list
```



![](/blog/post/images/pm2-list.png)



## Configuration



The config file is located at 



```bash
/Users/wuxiaopan/.config/verdaccio/config.yaml
```



### Storage

This is where we place all packages.

```yaml
# path to a directory with all packages
storage: ./store
```



### Uplinks

```YAML
# a list of other known repositories we can talk to
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
```



### Packages

Here, we define our own scope, which means that they can only be accessed internally. Other packages that are not started with your scope name will be downloaded from the official npmjs.

```yaml
packages:
  '@your-scope/*':
    # scoped packages
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    # proxy: npmjs
```



### Listen

The last step is to specify your ip and port.

```yaml
listen: 0.0.0.0:4873
```



Restart your server and verdaccio. We are done.



### Notify



```yaml
notify:
  method: POST
  headers: [{'Content-Type': 'application/json'}]
  endpoint: yourhooks
  content: '{"content": {"text": "{{ publishedPackage }} has published"}, "msg_type":"text"}'

```





## Web Page

If everything works well, open `http://localhost:4873/` and you will see the page like the below one.



![](/blog/post/images/verdaccio-web.png)





## Publish your package

Once your package is done, you can publish it to your private registry.



### add user

The first step is to register. The below command will ask you to input your name, password, and email.



```bash
npm adduser --registry http://localhost:4873/ 
```



### delete user

The registered users are stored in the following file, so we simply remove the corresponding record if we want to delete a user.

```bash
~/.config/verdaccio/htpasswd
```



### publish

Since we're publishing our package to the private registry, we should explicitly specify where the private registry is. To avoid specifing mannually each time, we use `nrm` to manage registries.



```bash
npm i -g nrm

nrm add localnpm http://localhost:4873
nrm use localnpm

nrm ls
```



![](/blog/post/images/nrm-ls.png)



The above command set the default registry to `localnpm`, which allows us to download the private packages from the localhost and the public packages from other public sources, such as `npmjs`.



Then we can publish 

```bash
npm publish
```

