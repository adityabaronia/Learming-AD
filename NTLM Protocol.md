- This blog will not cover whats inside the tools used.
- Will make a different blog on tools
- This blog will majorly cover only the NTLN protocol understanding.
- Othe rblog will be on:
    - kerberos
    - NTLM relay
    - Pass the hash 

## Preliminary

- **NT hash** and **LM hash** are the hashed version of user passwords. LM hash is totally obsolete. NT hash is commonly called as NTLM hash(Which is wrong). Thus whenwe talk about the user's password pash, we will refer to it as NT hash.
- NTLM(Windows New Technology LAN Manager) is therefore the name of the **authentication protocol**. it exist in 2 versions. NTLMv1 and NTLMv2.



# NTLM Protocol

The **NTLM protocol is authentication protocol used in Microsoft envirnoments**. In particular, it allows a user to prove their identity to a server in  order to use a service offered by this server.

<u><span style="color: green">**There are two prossible scenarios**</span></u> :
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
<span style="color: green">client uses a hashed version of thier pasword as a key for the following reasons: To avoid storing thier passwords in the clear tet on the sersver. <u>it's the password hash that is stored instead.</u> This hash is now the NT hash, which is nothing but the result of the MD4 function.</span>

<span style="color: green">To summarise, when the client authenticate, it uses the MD4 fingerprint of its password to encrypt the challenge. let's then see what happens on the server side once this responce is received.</span>

## Authentication
As explained earlier, there are two different scenarios. the first is that the <u>account used for authentication is local account, so the server has knowledge of this account, and it has a copy of the account's secret</u>. The second is that the <u>domain account is used, in which case the server has no knowledge of this account or its secret. <span style="color: green">it will have to delegate autherntication to the domain controller</span></u>.

## Local account
In the case where authentication is done with a local account, the server will encrypt the challenge it sent to the client with the user's secret key, or rather with the MD4 hash of the user;s secret. It will then check if the result of its opertation is equal to the client's response, proving that the user has the right secret. If not, the key used by the user is not right one since the challenge's encryption does not give the expected one.

In order to perform this operation, the server needs  to store the local users and the hash of thier password. The name of this database is the <span style="color: green">SAM(Security Account Manager)</span>. The SAM can be foundin the registry, especially with the regedit tool, but only when accessed as SYSTEM. It can be opened as SYSTEM with psexec:
```bash
psexec.exe -i -s regedit.exe
```
SAM can be fonud at:
```
Computer\HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users
```
<span style="color: green">A copy id also on disk in **C:\Windows\System32\SAM** So it contains the list of local users and their passwords, as well as the lsit of the local groups. well, to be more precise, it contains an ecrypted version of the hashes. But as all the information needed to decrypt them is also in registry (SAM and SYSTEM).</span>


To see how decryption mechanism for NT hash work, give a <span style="color: green">look into secretsdump.py[3] code or Mimikatz code[4]. </span>
SAM and SYSTEM databases can be backed up to extract the user's hashed passwords database.
First we save the two databases in a file
```bash
reg.exe save hklm\sam.save
reg.exe save hklm\system system.save
```
Then, we can use secretsdump.py to extract these hashes
```bash
secretsdump.py -sam sam.save -system system.save LOCAL
```

### Summary of verification process of Local Account

<span style="color: red">Since the server sends a challenge and the client encrypts this challenge with the hash of its secret and then sends it back to the server with its username , the server will look for the hash of the user's passwrod in its SAM database. Once it has it, it will also encrypt the challenge previously sent with this hash, and copmare its result with the one returned by the user. If it is the same then the user is authenticated! Otherwise, the iser has not provided the correct secret.</span>

## Domain account
When an authentication is done with a domain account, the user's NT hash is no longer stored on the server, but on the domain controller. The server to which user wants to authenticate receives the answer to its challenge, but it is not able to check if this answer is valid. It will delegate this task to the domain controller.

To do this, it will use the **Netlogon** service, which is able to establish a secure connection eith domain contrller, This secure connection is called **Secure channel**. This secure connection is possible because the server knows its own password, and the domain controller knows the ash of the server;s password. They can safely exchange a session key and communicate securly.

Server will send different elements to the domain controller in a structure called NETLOGON_NETWORK_INFO. Data inside this structure will be:
- The client's username (Identity)
- The challenge previously sent to the client(LmChallenge)
- The response to the challenge sent by the client (NtChallengeResponse)

The domain controller will look for the user's NT hash in its database. For the domain controller, it's not in the SAM, since it's domain account that tries to authenticate. This time it is in a file called **NTDS.DIT**, which is the database of all domain users. Once the NT hash is retrieved, it will compute the expected response with this hash and the challenge, and will compare this result with the client's response.

A message will then be sent to the server (NETLOGON_VALIDATION_SAM_INFO4) indicating wheather or not the client is authenticated, and it will also send a bunch of information about the user. This is the same information that is found in the PAC(PAC is in a way an extension of the Kerberos protocol used by Microsoft for the proper management of rights in an Active Directory) when kerberos authentication is used.

### Summary of verification process of Domain controller
<span style="color: red">The server sends the challenge to the client and the client encrypts this challenge with the hash of its secret and send it back to the sever along with its username and the domain name. This time the server will send this information to the domain controller in a secure Channel using Netlogon service. Once in possession of this information, the domain controller will also encrypt the challenge using the user;s hash, found in its NTDS.DIT database, and will then be able to compare its result with the one returned by the user. If it is same then the user is authenticated. Otherwise, the user has not provided the right secret. in both the cases, the domain controller transmits the information to the server.</span>

## NT hash limitations
From here we have understood that the plain text password is never used in exchanges, but the hashed version of the password called NT hash. It's simple hash of the plain text password.

If we think about it, stealing the plaintext password or stealing the hash is exactly the same. Since it is the hash that is used to respond to the challenge/Response, being in possession of the hash allows one to authenticate to a server. Having the password in clear text not useful at all.

<span style="color: green">We can say that having the NT hash is same as havign the password in clear text, in majority of the cases.</span>








## Reference
1. https://en.hackndo.com/pass-the-hash/#protocol-ntlm
2. https://en.hackndo.com/ntlm-relay/
3. **secretdump.py:** <span style="color: yellow">https://github.com/fortra/impacket/blob/master/impacket/examples/secretsdump.py#L1124</span>
4. **Mimikatz:** <span style="color: yellow">https://github.com/gentilkiwi/mimikatz/blob/master/mimikatz/modules/kuhl_m_lsadump.c</span>
5. https://github.com/fortra/impacket/blob/master/examples/psexec.py
