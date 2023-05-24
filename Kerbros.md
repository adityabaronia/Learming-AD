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
2. Ticket-Granting Service (TGS): He must then request a ticket to access the choosen service (e.g; CISF, HTTP, SQL, ...)
3. Application Request(AP): He finally uses the service by providing the ticket.

<span style="color: yellow">It is kind of like when you go to party. you have your ID that you had made and that proves that you are who you say you are (TGT). If you want to drink something, you have to go to the cashier with this ID (TGT) to ask for a drink ticket (TGS). The cashier wiil then give you a stamped, non-forgeable drink ticket with your age on it. Once you have it. you can go to the bar and ask for your drink by presenting the ticket. the bar can check that the ticket comes from the cash desk(thanks to the stamp), and serves you a good ol' fresh beer if you are not underaged.</span>

All right, but how does it work in practice?

We are in AD context, so the KDC(Key Distribution Center) is also the domain controller(DC). <u>The KDC contains all the domain information, including the secrets of each service, machine, user.</u> Thus, except for the DC, everyone only knows his own secret, and therefore do not know the secret of the other object in AD.

Let understand this from an example:

Let's take pixir as our user. <span style="color: yellow"><u>he wants to use a given service. to do so, he will need to authenticate himself to the KDC(Key Distribution Center) and he will  then send request to use the service. This phase is called **Authentication Service**</u><span style="color: yellow">


## Authentication Service
**KRB_AS_REQ** (Kerberos Authentication service request)
pixis will first send a request for a Ticket Grantinting Ticket(TGT) to the DC(Domain Controller). This request is called <u>KRB_AS_REQ(Kerberos Authentication Service Request)</u>. The TGT(Ticket Granting Ticket) represented by the client is a piece of encrypted information containing, among other things, a session key and a user information.

In order to perfrom this TGT request, <span style="color: yellow">pixis will send its name to KDC as well as the exact request's time, encrypted with hashed version of his password</span>.

The KDC will receive this username, and will verify that it exists in its database.

If it finds it,  it will then retrieve pixis hashed password which it will use to try to decrypt encrypted timestamp. if it can't, then the client didn't use the correct password to encrypt this timestamp. 
If it does, however, the KDC is assured that it is really pixis who is talking to him. It will generate a unique **session key** tied to this user and limited in time.

## Session Key

**KRB_AS_REP** (Kerberos Authentication service response)
The KDC will send back different things to pixis (KRB_AS_REP).
- <u>The session key, encrypted with pixis hashed password;</u>
- <u>The TGT, containing various information like:</u>
    - <u>Username</u>
    - <u>validity period</u>
    - <u>generate session key</u>
    - <u>The PAC(Privilege Atribute Certificate) which contains a lot of specific information about the user, including hia identifier (SID) and all the groups he is member of.</u>

<span style="color: red">TGT will be encrypted with the KDC key</span>. Thus, only the **KDC is able to decipher and read this ticket's content.**

Note that this TGT is considered public information. It can very well be intercepted durring the authentication phase. We will see in the following paragraph the importance of the authenticator that accompanies the TGT when the client communicates with the KDC.

The client receives these piece of information. Using his hashed password, the first part will be decrypted in order to retrieve the session key that will be necessary for further exchanges.

## Ticket-Granting Service(TGS)
Now that the user is authernticated, we are in the following situation: The user has his own key as well as a time-limited session key that only he currently knows, and a KDC-encrypted TGT that contains, among other things, this same session key.

**KRB_TGS_REQ** (Kerberos Ticket granting service request)
If pixis wants to use a service, e.g CISF on server01, it will send several piece of information to the KDC so that the KDC can send back a Service Ticket.
- the TGT;
- The service he wants to use and the associated host, so CISF/SERV01 in this example;
- An **authenticator**, which contains his username and current timestamp, all encrypted with the session key.

The authenticator is sent to make sure that it is pixis that is making the request, To do this, the KDC will compare TGT and authenticator contents. Since only the KDC can read TGT content, it could not have been tampered with. The KDC will read TGT content, including the TGT's owner and the associated session key. Then it will decrypt the authenticator content with the same session key. if the decryption works, and the data in the authenticator matches the data in the TGT, then pixis is who it claims to be. The KDC is assured that whoever made the request has the TGT and knowledge of the negotiated session key.


**KRB_TGS_REP**
Now that the KDC has been able to verify that the user is pixis, it send back information that will allow the userto make a request to the service. This message is the **KRB_TGS_REP**. 

<u>It(TGS) includes the following elements:</u>
- <u><span style="color: yellow">A ticket containing the name and host of the requested service</span>(CIFS/SERV01), user's username (pixis), the PAC, and a new session key which is only valid for communications between pixis and SERVER01 for certain period of time. <span style="color: red">This ticket is encrypted with the service's key</span> (i.e. the host key, since CISF service runs under the host account;)</u>
- <u><span style="color: yellow">The new session key</span></u>

These two pieces of information(the ticket and the session key) are encrypted with the first session key, the one that was initially exchanged between the KDC and the client.

The client will receive this message, and will be able to decrypt the first layer to get the session key created for communication with the service, as well as the ticket generated to use this service. This ticket is commonly called **Ticket-Granting-Service(TGS)**.


## Application Request (AP)
**KRB_AP_REQ** (kerberos application request)
pixis will then generate a **new authenticator** which it will **encrypt with this new session key**(got in TGS), **along with the TGS**. this is same process as with the KDC.

CIFS service receives the TGS and can decrypt it with its own secret. Since only the DCS knows its secret, he rests assured that this TGS is authentic. This TGS contains the session key it will use to decrypt the authenticator. By comparing the TGS and authenticator contents, the service can be certain of the user's authenticity.




## Reference
1. https://en.hackndo.com/kerberos/
2. https://en.hackndo.com/kerberoasting/
