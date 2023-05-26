# Service Principal Name

This documentation focuses on SPN (ServicePrincipal Names) in order to understand what they are and how they are used.

## What is an SPN
We are in an Active Directory Environment. To understand what is SPN, we must understand what the notion of service within an Active Directory is.

A service is actually  feature , of a software, something that cn be used by other members of the AD. You can have for example a web server, a network share, a DNS service, a printing service, and so on. To identify a service, we need at least two things. The same service can run on different hosts, so we need to specify the host, and a computer can host several services, so we need to specify the service, obviously.

It is by combining these information that we can accuratley designate a service. This combination represents its Service Principal Name(SPN). It looks like this:

`service_class/hostname_or_FQDN`

The service class is actually a somewhat generic name for the service. For exampl, all web servers are grouped in the "www" class and SQL services are in the "SqlServer" class.

If the service runs behind a custom port, or if you want to specify it to avoid any ambiguity, you can append it to the hostname:

`service_class/hostname_or_FQDN:port`

Optionally, you can name a SPN.

`service_class/hostname_or_FQDN:port/arbitray_name`

For example, in Active Directory, we have two hosts offering web services, WEB-SERVER-01 and WEB-SERVER-02, and each of the other two machines offer other services.

If I want to designate the web server on WEB-SERVER-01, the SPN looks like this:

`www/WEB-SERVER-01`

or 
ticket was created after someone asked for `www` service om `WEB-SERVER-01` in `adsec.local` domain.

`www/WEB-SERVER-01.adsec.local`


There is a special case that we encounter in SPN attributes of an object in AD, it is the `HOST` SPN.

`HOST` SPN is not really a service class. It’s a group of service classes, a kind of alias that groups together a large number of SPNs. The elements it groups together are defined in the Active Directory’s “SPN-Mappings” attribute. These classes can be listed with the following command:

`Get-ADObject -Identity "CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration,DC=HALO,DC=NET" -properties sPNMappings`