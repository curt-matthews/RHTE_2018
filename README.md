# RHTE 2018: Hybrid Cloud Automated Provisioning


## (Outline and Documentation Links)


# Abstract:

If you follow the below steps on a green-field, configured, Red Hat CloudForms appliance(s), Hybrid Cloud Provisioning with Infrastructure Providers (RHEV, vCenter), a Configuration Management Provider (Satellite) and an Automation Provider (Ansible Tower), becomes a reality.  Simply follow the steps listed below and "it just works".

# Assumptions:

*   Red Hat CloudForms 4.5/4.6 up and running…
*   Red Hat Satellite 6 configured…
*   Ansible Tower setup (for post install configuration)...
*   Infra Providers and networking are in place.

# Additional Requirements:

## TL;DR - Assign an email address to the admin user.

*   Each user, including the admin user, making provisioning requests must have an email address configured.  Provisioning requests will fail if initiated by any user without a configured email address.  The [General Configuration](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/general_configuration/configuration) document explains the process of creating users and editing users.

# Configure Providers:

*   Add Infrastructure, Configuration and Automation Providers as prescribed in [Managing Providers](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/managing_providers)

## Provider Connection Info

| Provider Name          | Provider Type             | URL                  | Username       | Password
|------------------------|---------------------------|----------------------|----------------|----------
| Red Hat Virtualization | Infrastructure            | rhvm.example.com     | admin@internal | r3dh4t1!
| VMware vCenter         | Infrastructure            | vcenter.example.com  | root           | r3dh4t1!
| Red Hat Satellite      | Configuration             | sat.example.com      | admin          | r3dh4t1!
| Ansible Tower          | Automation: Ansible Tower | ansible1.example.com | admin          | r3dh4t1!

## Connect Infrastructure Providers:

*   Connect VMware vCenter
*   Connect RHEV

## Add Configuration Management Provider:

*   Add Red Hat Satellite Provider

## Connect Automation Provider:

*   Connect Ansible Tower Provider

# General Configuration:

*   The Git Repositories Owner Role is disabled by default on a newly installed CloudForms appliance.  This role will need to be enabled in order to import the upstream Automate Domains.  Follow the guidelines in [General Configuration](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/general_configuration/configuration) in order to complete these steps.

## Enable and configure Git Repositories Owner Server Role:
  *   Enable 'Git Repositories Owner' Server Role
  *   Promote 'Git Repositories Owner' Role to Primary if necessary
  *   Start the 'Git Repositories Role' if necessary

# Automate Domains:

## Import Upstream Domains:

*   [Understanding the Automate Model - Importing a Domain](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/scripting_actions_in_cloudforms/understanding-the-automate-model#importing-a-domain) explains how to import the required Automate domains.  The relevant repositories with links are included below.  In order to control release, you may want to choose a specific branch or tag.

### Import Automate Domains from RedHatOfficial Github:

*   Import miq-Utilities
    *   [https://github.com/RedHatOfficial/miq-Utilities.git](https://github.com/RedHatOfficial/miq-Utilities.git)
		*   For the purposes of this lab, select Tag `v9.1`
*   Import miq-RedHat-Satellite6
    *   [https://github.com/RedHatOfficial/miq-RedHat-Satellite6.git](https://github.com/RedHatOfficial/miq-RedHat-Satellite6.git)
		*   For the purposes of this lab, select Tag `v8.5`

## Create Configuration Domain:

*   [Understanding the Automate Model - Creating a Domain](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/scripting_actions_in_cloudforms/understanding-the-automate-model#creating-a-domain) covers the steps required to create a domain within the Automate model.  The recommendation is to create a domain with a name of Configuration but the name of the domain is inconsequential.  The ordering of the domains is important and that will be covered later in this document.

### Create Configuration Domain:

*   Create Configuration Domain and Enable

### Network Configuration:

*   Override RedHatConsulting_Utilities/Infrastructure/Network/Configuration by copying the Class into your new configuration domain.
    *   Add a new instance for each provisioning and destination VLAN
    *   RHV Network Settings
        *   Name: pxenet
        *   (network purpose): [provisioning, destination]
        *   (network_address_space): 10.10.10.0/24
        *   (network_gateway): 10.10.10.1
        *   (network_nameservers: [192.168.0.1]
        *   (network_ddi_provider): satellite
    *   vCenter Network Settings
        *   Name: VM_Network
        *   (network purpose): [provisioning, destination]
        *   (network_address_space): 10.10.10.0/24
        *   (network_gateway): 10.10.10.1
        *   (network_nameservers: [192.168.0.1]
        *   (network_ddi_provider): satellite

## Automate Domain Priority:

*   [Understanding the Automate Model - Changing Priority Order of Domains](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/scripting_actions_in_cloudforms/understanding-the-automate-model#changing-priority-order-of-domains) explains the process required in order to properly configure the priority of Automate Domains.  The following is the necessary priority required for this configuration.

### Configure Automate Domain Priority:

*   Configuration
*   RedHatConsulting_Satellite6
*   RedHatConsulting_Utilities

# Service Dialogs and Catalogs:

*   Pre-configured catalog and associated service dialogs are provided as part of the miq-RedHat-Satellite6 project on Github.  Complete the following steps to import these items.

*  For the purpose of this lab, the cfme-rhconsulting-scripts repository has been cloned to `/root/src/cfme-rhconfulting-scripts/` and installed per the instructions in the link below.  Installation of these scripts provides the miqimport command required in the next steps.
    *   [https://github.com/rhtconsulting/cfme-rhconsulting-scripts#install](https://github.com/rhtconsulting/cfme-rhconsulting-scripts#install)

## Import Service Dialog and Catalog Items:

*   We have cloned tag v8.5 of the miq-RedHat-Satellite6 repository to `/root/src/miq-RedHat-Satellite6` on cf.example.com for your convenience.
    * The repository contains dialogs for versions CFME 4.5 (5.8) and 4.6 (5.9), the latter of which is being used for this lab.
*   Import Service Dialog
    *   ```miqimport service_dialogs /root/src/miq-RedHat-Satellite6/Dialogs/v5.9```
*   Import Catalog
    *   ```miqimport service_catalogs /root/src/miq-RedHat-Satellite6/Catalogs/v5.9```

# Tagging:

## Tag Categories:

### Environment Tag Category

*   Delete environment Tag Category
*   Recreate environment Tag Category
    *   Name: environment
    *   Description: Environment
    *   Long Description: Environment
    *   Show in Console: Yes
    *   Single Value: No

### Operating System Tag Category

*   Create os Tag Category
    *   Name: os
    *   Description: Operating System
    *   Long Description: Operating System
    *   Show in Console: Yes
    *   Single Value: Yes

## Tags:

### Auto-Populate Satellite Tags

*   In order to auto-populate some of the tags, you can simply open the service dialog at this point.  Once the dialog has opened, simply close the dialog and continue with tagging.  This will create the appropriate environment and location tags based on your Satellite configuration.

### Operating System Tags:

*   Create Tag for each Operating System Supported
    *   i.e. name: rhel7, description: RHEL 7

### Infrastructure Provider Tags:

*   Infrastructure Providers require a multitude of tags based on your organization's configuration and design.  The providers being used for automated provisioning need the following tags applied appropriate to your organization's goals.
*   Location Tags
    *   Tag each Provider with appropriate location tag
		    *    VMware vCenter - vCenter
		    *    Red Hat Virtualization - RHV
*   Environment Tags
    *   Tag both Providers with ```Dev``` tag
*   Provisioning Scope: All
    *   Navigate to Compute / Infrastructure / Hosts
        *   For the purposes of this lab, tag all available hosts
    *   Navigate to Compute / Infrastructure / Datastores
        *   For the purposes of this lab, tag ```vmstore00``` and ```vsphere1-datastore```
*   Tag Templates
    *   Navigate to Compute / Infrastructure / Virtual Machines
    *   Select Templates
    *   Select the ```rhel7_diskless``` template to be tagged for ```VMware vCenter``` and ```Red Hat Virtualization``` Providers
    *   Select Policy / Edit Tags
    *   Apply ```RHEL 7``` Operating System Tag
    *   Save

# The Proof is in the Pudding…

*   Now is the time to test your configuration.  Ensure you have your logs open in case of configuration errors.
*   Open the Service Dialog
    *   If any script errors exist, refer to logs to determine the issue
*   Populate the Dialog
    *   The Infrastructure tab of the form will change as entries are populated.  Use this to verify your selections.
*   Submit the Order
*   Watch the Requests and the Logs
*   Rejoice as you see your Hybrid Cloud Automated Provisioning Realized
    *   Do a little dance
    *   Tell all of your friends
*   Get involved!
    *   Submit issues against the upstream projects
    *   Pull Requests
    *   Pull Requests
    *   Pull Requests

*  To find the complete installation document which includes configuring a Cloud Provider (AWS), please refer to [https://github.com/RedHatOfficial/miq-RedHat_Satellite6/INSTALL.md](https://github.com/RedHatOfficial/miq-RedHat_Satellite6/INSTALL.md)
