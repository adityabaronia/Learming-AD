
# Introduction

NTLM relay relies, as its name implies, on NTLM authentication. The basics of NTLM have been presented in pass-the-hash writeup. As a reminder, NTLM protocol is used to authenticate a client to server. what we call client and server are two parts of the exchange. The client is the one that wishes to authenticate itself, and the server is the one that validates this authentication.

This authentication takes place in 3 steps:
1. First the client tells the server that it want to authenticate.
2. The server than respondes with a challenge which is nothing more than a random sequence of characters.
3. The client encrypts this challenge with its secret, and sends the result back to the server. This is its response.

This process is called **Challenge-Response**.

# NTLM relay

Scenarion of NTLM relay attack goes like: An attacker manages to be in a main-in-the-middle position means that from a clients POV, the attacker's machine is the server to which he wants to authenticate, and from the server's point of view, the attacvker is a client like anyother who wants to authenticate.

Except that the attacker does not "just" want to authenticate to the server. He wishes to do so by pretending to be the client. However, he does not know the secret of the cleint, and even if he listens to the conversations, as this secret is never transmitted over the network, the attacker is not able to extract any secret. So, how does it work.

## Message Realying

During NTLM authenticaytion, a client can prove to a server its identity by encrypting with its password some piece of information provided by the server. So the only thing the attacker has to do is to lewt the client do its work, and passing the message from the client to the server, and the replies from the server to the client.

All the cleint has to send to the server, the attacker will receive it, and he will send the message back to the real server, and all the message that the server sends to the client, the attacker will also receive them, and he will forward them to the client, as is.

At the end of these exchanges, the attacker is authenticated on the server with the client’s credentials.

