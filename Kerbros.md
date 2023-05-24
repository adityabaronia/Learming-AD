# Kerberos in Active Directory

**AD is a microsoft solution used for <u>Windows network management</u>**, and provides the following services:
- Directory service (LDAP)
- Authentication (Kerberos)
- Name Resolution (DNS)
- Homogeneous software policy

In this blog, we will focus on the authentication part within Active Directory, based on Kerberos.

<u>Kerberos is a protocol that allows users to authenticate on the network, and access services once authenticated.</u>

## How it works

Kerberos is used whenever a user wants to access some services on the network. Thanks to kerberos the user won't need to type his password every time and the server won't need to know every user's password. **This is centralized authentication**.

In order to do this, at least three entites are required
- A client
- A service
- A key Distribution Ceneter(KDC) which is Domain Controller(DC) in active directory environment.

The idea is that when a client wants to access a service, no password will be sent over the network, thus avoiding password leaks that could compromise the network.

For this the process is bit cumbersome, and is divided into three steps:
1. Authentication Service(AS): The client ust authenticated itself to KDC(key distribution center).
2. Ticket-Granting Ticket (TGT): He must then request a ticket to access the choosen service (e.g; CISF, HTTP, SQL, ...)
3. Application Request(AP): He finally uses the service by providing the ticket.

<span style="color: yellow">It is kind of like when you go to party. you have your ID that you had made and that proves that you are who you say you are (TGT). If you want to drink something, you have to go to the cashier with this ID (TGT) to ask for a drink ticket (TGS). The cashier wiil then give you a stamped, non-forgeable drink ticket with your age on it. Once you have it. you can go to the bar and ask for your drink by presenting the ticket. the bar can check that the ticket comes from the cash desk(thanks to the stamp), and serves you a good ol' fresh beer if you are not underaged.</span>

All right, but how does it work in practice?

We are in AD context, so the KDC(Key Distribution Center) is also the domain controller(DC). <u>The KDC contains all the domain information, including the secrets of each service, machine, user.</u> Thus, except for the DC, everyone only knows his own secret, and therefore do not know the secret of the other object in AD.



##Reference
1. https://en.hackndo.com/kerberos/