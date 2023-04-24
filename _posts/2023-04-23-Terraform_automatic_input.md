---
title: Terraform automatic input
date: 2023-04-23 17:00:00 -0000
categories: [Terraform]
tags: [aws, terraform, json]
---

# Terraform automatic inputs

I build my test or development environments on AWS using Terraform, to keep them reasonably secure I include a list of IP's in the ingress list for the security group. One of those IP's is the IP I'm coming in from, this can change if either my laptop or router get rebooted so it feels kinda wrong to include this config statically in my Terraform code, and it's not something I really want pushed up to Github. So I need a way of dynamically getting my IP every time I run ```terraform apply```.  Fortunately, this can be done with the [external](https://registry.terraform.io/providers/hashicorp/external/latest) provider.
The [external](https://registry.terraform.io/providers/hashicorp/external/latest) provider is designed to allow the integration of other applications via an API, so unsurprisingly, it's expecting some json.

So the first thing to do is write a bash script that'll find my IP address and return it as a json object.

```bash
#!/bin/bash

ext_ip=`curl -s https://freedns.afraid.org/dynamic/check.php | grep REMOTE_ADDR |  awk '{print $3}'`
echo "{ \"my_ip\" : \""$ext_ip"/32\" }"
```

This just runs a curl to [freedns.afraid.org](https://freedns.afraid.org/) and returns my network info, I'll use a grep and an awk to extract my IP address from that. The last line returns that IP address formatted as some json. Now I need to get Terraform to use it, to do that I'll add this to my *main.tf*.

```hcl
data "external" "myip" {
  program = [ "bash", "./get_my_ip.sh"]
}
```

This is where the external provider comes in, we've told it to use a bash script and the bash script location relative to the ```main.tf``` file. Now we just need to pass that data as a parameter when I call my network module.

```hcl
allow      = [ data.external.myip.result.my_ip ]
```

The ```allow``` parameter is expecting an array, it does usually have a few other IP's in there but I've left those out for clarity. The important thing is that you can see how the name of that data is formed.

And that's it, when I run ```terraform apply``` my AWS environment will be configured to let my laptop connect and if, for any reason, my router or laptop reboots and it's IP address changes, I just run ```terraform apply``` again to update the AWS environment with the new IP address.