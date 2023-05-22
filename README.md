# NTLMRelay-TechnicalDetails

# NTLMRelay-TechnicalDetails

NTLM relay is a technique of standing between a client and a server to perform actions on the server while while impersonation the client.
it can be very powerful and can be used to take control of an active Directory domain from a black box(no credentials)

## NTLM Protocol

The **NTLM protocol is authentication protocol used in Microsoft envirnoments**. In particular, it allows a user to prove their identity to a server in  order to use a service offered by this server.

<u><<span style="color: green">**There are two prossible scenarios**</span></u> :
- <span style="color: green">Either the user uses the credentials of a local account of the server, in which case the server has the user's secret in its local database amd will be able to authenticate the user;</span>
- <span style="color: green">Or in an Active Directory environment, the user uses a domain account during authentication, in which case the server will have to ask the domain controller to verify the information provided by the user.</span>

In both cases, authentication begins with a **challenge/response** phase between the client and the server.

## Challenge-Response
The <u>challenge/response principle is used so that the server verifies  tht the user knows the secret of the account he is authentication with</u>, without passing the password through the network. This is called zero knowledge proof.
1. <u><span style="color: green">Negotiation</span></u> :The client tells the server that it wants t authenticate to it(NEGOTIATE_MESSAGE).
2. <u><span style="color: green">Challenge</span></u> : The server sends the challenge to the client. This is nothing more than a 64-bit valuethat cahanges with each authentication request(CHANLLENGE_MESSAGE).
3. <u><span style="color: green">Response</span></u> : The client encrypts the previously received challenge using a hashed version of its password as the key, and returns this encypted version to the server, along with its user name and possibly its domain(AUTHENTICATE_MESSAGE). 

In this communication between server and client, ther server is in possiotion of two things:
1. The challenge it sent to client
2. The client's response that was encryptred with his(client's) secret

<u>To finalize the authentication, the server only has to check the validity of the response sent by the client. But just before that, let's do a little check on the client's secret.</u>

## Authentication secret
client uses a hashed version of thier pasword as a key for the following reasons: To avoid storing thier passwords in the clear tet on the sersver. <u>it's the password hash that is stored instead.</u> This hash is now the NT hash, which is nothing but the result of the MD4 function.

<span style="color: green">To summarise, when the client authenticate, it uses the MD4 fingerprint of its password to encrypt the challenge. let's then see what happens on the server side once this responce is received.</span>

## Authentication
As explained earlier, there are two different scenarios. the first is that the <u>account used for authentication is local account, so the server has knowledge of this account, and it has a copy of the account's secret</u>. The second is that the <u>domain account is used, in which case the server has no knowledge of this account or its secret. <span style="color: green">it will have to delegate autherntication to the domain controller</span></u>.

## Local account
In the case where authentication is done with a local account, the server will encrypt the challenge it sent to the client with the user's secret key, or rather with the MD4 hash of the user;s secret. It will then check if the result of its opertation is equal to the client's response, proving that the user has the right secret. If not, the key used by the user is not right one since the challenge's encryption does not give the expected one.

In order to perform this operation, the server needs  to store the local users and the hash of thier password. The name of this database is the SAM(Security Account Manager). The SAM can be foundin the registry, especially with the regedit tool, but only when accessed as SYSTEM. It can be opened as SYSTEM with psexec:
```bash
psexec.exe -i -s regedit.exe
```
SAM can be fonud at:
```
Computer\HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users
```
A copy id also on diskin C:\Windwos\System32\SAM

So it contains the list of local users and their passwords, as well as the lsit of the local groups. well, to be more precise, it contains an ecrypted version of the hashes. But as all the information needed to decrypt them is also in registry (SAM and SYSTEM).





## Preliminary

- **NT hash** and **LM hash** are the hashed version of user passwords. LM hash is totally obsolete. NT hash is commonly called as NTLM hash(Which is wrong). Thus whenwe talk about the user's password pash, we will refer to it as NT hash.
- NTLM(Windows New Technology LAN Manager) is therefore the name of the **authentication protocol**. it exist in 2 versions. NTLMv1 and NTLMv2





## Reference
- https://en.hackndo.com/pass-the-hash/#protocol-ntlm
- https://en.hackndo.com/ntlm-relay/
