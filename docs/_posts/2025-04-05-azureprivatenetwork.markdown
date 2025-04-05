---
layout: post
title:  "Connect azure mysql via virtual machine"
date:   2025-04-04 23:00:00 +0000
categories: azure bastion mysql server ssh
---

Connecting to an Azure MySQL Server through a virtual machine (VM) instead of directly can have several advantages.

One of the greatest benefits is security. Your database server does not expose any api
to public world. Therefore, since there is no public ip, it is by default secure and protected against malicious users.



{% highlight bash %}
#!/usr/bin/env sh
BASTION=
BASTION_RG=
SUBSCRIPTION_ID=
RESOURCE_GROUP=
VM_NAME=
REMOTE_PORT=
SSH_PRIVATE_KEY=
LOCAL_PORT=
DATABASE_SERVER=

az network bastion tunnel --name $BASTION --resource-group $BASTION_RG \
    --target-resource-id /subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Compute/virtualMachines/$VM_NAME \
    --resource-port "22" \
    --port $REMOTE_PORT --subscription $HUB_SUB

{% endhighlight %}

It is worth to explain what the script is doing. This <b>az network bastion tunnel</b> command is used to create a tunnel through Azure Bastion to securely connect to a Virtual Machine (VM)—usually via SSH on port 22. 

We first set up a tunnel to connect to virtual machine. You can think of a bastion as a path to virtual machine. Once you set up a tunnel,
you are a part of the same virtual network as virtual machine.

| Syntax                              | Description                                                                                                                                                        |
|-------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <i>az network bastion tunnel</i>    | This Azure CLI command sets up a Bastion tunnel, allowing local access to the VM via Bastion without needing a public IP on the VM.                                |
| <i>--name $BASTION</i>              | The name of the Azure Bastion resource you're using                                                                                                                |
| <i>--resource-group $BASTION_RG</i> | The resource group where the Bastion resource resides.                                                                                                             |
| <i>--target-resource-id</i>         | This is the full Azure Resource ID of the <b>VM you want to connect to</b>. You provide the subscription ID, resource group, and VM name to specify the target VM. |
| <i>---resource-port "22"</i>        | The port on the VM you want to connect to. Port 22 is standard for SSH.                                                                                            |
| <i>--port $REMOTE_PORT</i>          | This is the <b>local port on your machine</b> where the tunnel will be established. You'll connect to localhost:$REMOTE_PORT to access the VM.                     |
| <i>--subscription $HUB_SUB</i>      | Specifies which Azure subscription contains the Bastion resource (could be different from the VM’s subscription).                                                  |

The next command that forwards requests that is being sent by your local port to mysql server. Since you are in a same virtual network, connect to mysql is a similar like it is being installed locally.

Once you execute ssh commands: 

{% highlight bash %}

ssh -p $REMOTE_PORT azure@127.0.0.1 -NL $LOCAL_PORT:DATABASE_SERVER:3306 -i $SSH_PRIVATE_KEY

{% endhighlight %}

- Any connection to that local port is forwarded securely through Azure Bastion to port 22 on the target VM.
