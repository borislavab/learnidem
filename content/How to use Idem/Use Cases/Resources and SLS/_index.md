---
title: "Resources and SLS"
weight: 55
---

In this section, we showcase how to link existing resources in your cloud environment to new ones that we will create via a [present state](/How-to-use-Idem/States/Present/)<br>
We will create a basic Azure Virtual Machine. For that, we will discover or [describe](/How-to-use-Idem/Describe/) an existing NIC, then assign it to a new Azure Virtual Machine resource in order to create it.

Later on, at the [End To End Azure VM Use case](/How-to-use-Idem/Use-Cases/End-to-End-Azure-VM/), you will be able to reference other and more types of resources.

In order to create a Azure VM, we will need to indicate the <b>networkInterfaces.id</b> property.

Of course, we could create the NIC also with the [Idem Azure Plug-In Resource](/Getting-Started/Cloud-Plug-Ins/Idem-Azure-Plug-In/) <b>"virtual_networks"</b>, but for the sake of this example, we already have an existing & un-assigned NIC interface that we could use in our cloud environment.

You can easily discover as follows, 

```shell
idem describe azure.virtual_networks.network_interfaces
```
Please note that you may have more than one NIC available, you can use the [Filter Flag](/How-to-use-Idem/Filter-flag/)
In this case, the output of the command shows our network interface available

```shell
--------
? /subscriptions/23a8cee7-a1e4-4bb3-aff9-6898b4ee6fde/resourceGroups/group8a87dd00ea7183a729588634c88e5123/providers/Microsoft.Network/networkInterfaces/default
: azure.virtual_networks.network_interfaces.present:
  - network_interface_name: default
  - resource_group_name: group8a87dd00ea7183a729588634c88e5123
  - parameters:
      etag: W/6f33dd08-64fa-4d11-8c17-2d9fdea97671
      "id: /subscriptions/23a8cee7-a1e4-4bb3-aff9-6898b4ee6fde/resourceGroups/group8a87dd00ea7183a729588634c88e5123/providers/Microsoft.Network/networkInterfaces/default"
      location: centralus
      name: default
....
```

Now, let's write a new state <b>"vm_moff_present.sls"</b> defining our Azure VM
that includes the idem [present](/Getting-Started/Basic-Commands/) directive for <b>azure.compute.virtual_machines</b>,<br> 
but also the specific NIC id described above (more about Azure Plug-In Resources ID in the [End to End Azure VM](/How-to-use-Idem/Use-Cases/End-to-End-Azure-VM/) section)

```shell
        networkProfile:
          networkInterfaces:
          - id: /subscriptions/23a8cee7-a1e4-4bb3-aff9-6898b4ee6fde/resourceGroups/group8a87dd00ea7183a729588634c88e5123/providers/Microsoft.Network/networkInterfaces/default
```

Of course, please note that we have included other properties needed for creating the Azure VM, such as VM Name, Resource Group, Hardware Profile (Flavor), ImageReference, etc.

```shell
  azure.compute.virtual_machines.present:
  - resource_group_name: moff-idem-01
  - vm_name: Development-idem-015042
  - parameters:
      location: centralus
      name: Development-idem-015042
      properties:
        hardwareProfile:
          vmSize: Standard_A5
        networkProfile:
          networkInterfaces:
          - id: /subscriptions/23a8cee7-a1e4-4bb3-aff9-6898b4ee6fde/resourceGroups/group8a87dd00ea7183a729588634c88e5123/providers/Microsoft.Network/networkInterfaces/default
            properties:
              primary: true
        storageProfile:
          dataDisks: []
          imageReference:
            exactVersion: 18.04.201804262
            offer: UbuntuServer
            publisher: Canonical
            sku: 18.04-LTS
            version: 18.04.201804262
        osProfile:
          adminUsername: azureuser
          allowExtensionOperations: true
          computerName: Development-idem-015042
          adminPassword: "C@n0n1c@lVMware"
          linuxConfiguration:
            disablePasswordAuthentication: false
            patchSettings:
              assessmentMode: ImageDefault
              patchMode: ImageDefault
            provisionVMAgent: true
          secrets: []
```

We now can simply execute it by calling the command "state"

```shell
idem state vm_moff_present.sls
```

This will return the following output (please note the provisioningState):

```shell
--------
      ID: Development-idem-015042
Function: azure.compute.virtual_machines.present
  Result: True
 Comment: Created
 Changes: new:
    ----------
    name:
        Development-idem-015042
    id:
        /subscriptions/23a8cee7-a1e4-4bb3-aff9-6898b4ee6fde/resourceGroups/moff-idem-01/providers/Microsoft.Compute/virtualMachines/Development-idem-015042
    type:
        Microsoft.Compute/virtualMachines
.....
```

Then you can search for the newly created Azure VM with [describe](/How-to-use-Idem/Describe/).
Please note that you may have more than one VM available - you can use the [Filter Flag](/How-to-use-Idem/Filter-flag/) to narrow the search.
In this case, the output of the command shows our single Azure VM

```shell
idem describe azure.compute.virtual_machines
```

It is worth to mention that some resources may have dependencies among them and be created at the same time. In that scenario you could include the [ reconciler=basic flag](/How-to-use-Idem/States/reconciler-flag/). This allows Idem-azure-auto to run [Idem state](/How-to-use-Idem/States/Present/)
with Idem's reconciliation loop.

