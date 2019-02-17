---
layout: post
title:  "HTTP Request/Response life cycle"
date:   2019-02-17 17:10:00 +0200
categories: [tech, web, http, dns]
---

A few years ago, a friend of mine from GitLab asked me a question.
Now this question is used as one of the main during the screening process.
There are my thoughts on it.

### Question

> A user browses to https://gitlab.com/gitlab-org/gitlab-ce in their browser.
Please describe in as much detail as you think is appropriate the lifecycle of this request and what happens
in the browser, over the network, on GitLab servers, and in the GitLab Rails application before the request completes.

### Steps

1. A user types the URL in the address field of their web browser and presses Enter

2. The browser parses an information contained in the URL. This includes:

	* the protocol = **https**
	* the domain name = **gitlab.com**
	* and the path (resource) = **/gitlab-org/gitlab-ce**

3. Now the browser needs to resolve the name _"gitlab.com"_ to an IP address and it asks an operating system for that.

4. _DNS lookup_ ([more about it below](#dns-lookup)).

5. Once the DNS client receives the IP address of the destination server, it sends it to the browser.

	Often it is a virtual IP address which points to some _Load Balancer_.

6. The browser takes the IP address and the given port number from the URL (the HTTPS defaults to port 443)

	example in my case.: _35.231.145.151:443_.

7. The browser initiates an _SSL/TLS handshake_ ([more about it below](#ssltls-handshake)) and opens a TCP socket connection.

8. At this point, the web browser and the web server (NGINX in our case) are finally connected.

9. The web browser sends an HTTP GET request to the web server for the desired resource:

	* HTTP verb = GET
	* host = gitlab.com
	* path = /gitlab-org/gitlab-ce
	* protocol & version = HTTP/1.1

10. NGINX acts as the first line reverse proxy and can fullfil some requests by itself. It accepts the request and sends it (if neccessarry) to a Unix domain socket

11. Upon which [GitLab Workhorse](https://gitlab.com/gitlab-org/gitlab-workhorse) process accepts it. Also can handle some requests without involving the app server at all and/or determines if it needs to go somewhere else.

12. Reqeust comes to one of Unicorn workers and passes through rack middleware (can hit other services) and all the way back

	- Rails Router -> Dispatcher -> Controller
	- Optionally: Postgres/Gitaly/Redis - depending on the type of request, it may hit these services to store or retreive data.

	The most interesting part here is [Gitaly](https://gitlab.com/gitlab-org/gitaly). It provides high-level RPC access to Git repositories. Without it, no other components can read or write Git data. GitLab (app server / workhorse) acts like a client to Gitaly server. Gitaly acts like a service for performing Git operations.

13. The web browser receives the HTTP response (with content type text/html in our case) and then parses through it doing a full head to toe scan looking for other assets that are listed, such as images, CSS files, JavaScript files, etc.

14. For each asset listed the browser makes an additional HTTP request to the server. Also additional requests can be made by frontend. (33 total in my case).

15. Once the browser has finished loading all other assets that were listed in the HTML page, the page will finally be loaded in the browser window ([more about it below](#browsers-rendering-engine)) and the connection will be closed.

### DNS lookup

Let's assume that neither client machine's nor router nor ISP nor any other server in a chain contains DNS cache for that name.

The are two types of queries that a DNS resolver (either a DNS client or another DNS server) can make to a DNS server. It is important to mention a difference between them now:

- In a **recursive query**, the queried name server is requested to respond with the requested data or with an error stating that data of the requested type or the specified domain name does not exist. The name server cannot just refer the DNS resolver to a different name server. A DNS client typically sends this type of query.

- In an **iterative query**, the queried name server can return the best answer it currently has back to the DNS resolver. The best answer might be the resolved name or a referral to another name server that is closer to fulfilling the DNS client's original request. DNS servers typically send iterative queries to query other DNS servers.

1. The DNS resolver on the DNS client sends a _recursive query_ to its configured DNS server, requesting the IP address corresponding to the name _"gitlab.com"_. The DNS server for that client is responsible for resolving the name and cannot refer the DNS client to another DNS server.

2. The DNS server that received the initial _recursive query_ checks its zones and finds no zones corresponding to the requested domain name; the DNS server is not authoritative for the _"gitlab.com"_ domain. Because the DNS server has no information about the IP addresses of DNS servers that are authoritative for _"gitlab.com."_ or _"com."_, it sends an _iterative query_ for _"gitlab.com."_ to a root name server.

	- _A trailing period designates a domain name of a host relative to the root domain. To connect to that host, a user would specify the name "gitlab.com." If the user does not specify the final period, the DNS resolver automatically adds it to the specified name._

3. The root name server is authoritative for the root domain and has information about name servers that are authoritative for top-level domain names. It is not authoritative for the _"gitlab.com."_ domain. Therefore, the root name server replies with the IP address of a name server for the _"com."_ top-level domain.

4. The DNS server of the DNS client sends an _iterative query_ for _"gitlab.com."_ to the name server that is authoritative for the _"com."_ top-level domain.

5. The _"com."_ name server is authoritative for the _"com."_ domain and has information about the IP addresses of name servers that are authoritative for second-level domain names of the _"com."_ domain. It is not authoritative for the _"gitlab.com."_ domain. Therefore, the _"com."_ name server replies with the IP address of the name server that is authoritative for the _"gitlab.com."_ domain.

6. The DNS server of the DNS client sends an _iterative query_ for _"gitlab.com."_ to the name server that is authoritative for the _"gitlab.com."_ domain.

7. The _"gitlab.com."_ name server replies with the IP address corresponding to the fully qualified domain name (FQDN) _"gitlab.com."_.

8. The DNS server of the DNS client sends the IP address of _"gitlab.com."_ to the DNS client.

![DNS lookup](https://cdn-images-1.medium.com/max/1600/0*sbTvDcrA3cKVJucF.gif)

### SSL/TLS handshake

1. The browser (client) is going to use the standart HTTP protocol but with an additional layer of encryption on top of it which provides with:

	- **authentication** - proving that the server the client is talking to is who it says it is.
	- **privacy** - encrypting data such that anything in-between the client and the server cannot read the traffic.
	- **integrity** - ensuring that the data received on either end has not been altered unknowingly along the way.

2. A few words about cryptography. Broadly speaking, there are two types of encryption schemes:

	- **symmetric** (or private key) - the same key is used to encrypt and decrypt a data
	- **assymetric** (or public key) - there are two keys. the public key and the private one
		- **public key** - can be openly distributed. is used for:
			1. verification that an owner of the paired private key sent the message (verification of a digital signature)
			2. encryption of the message
		- **private key** - must be kept absolutely private by the owner. is used for:
			1. decryption of the message encrypted with the paired public key
			2. digital signing of the messages

	SSL/TLS handshake uses the assymetric one.

3. With all that said, GitLab Server's HTTPS Certificate can be simply called a public key. So, to talk securely to GitLab I just need to know its public key, and this is true for any other website. And here is a problem. It is impossible to keep an updated list of public keys for every single website on a client machine. This approach does not scale. It is also not possible to pull the public key during the first connect. Because there is not a secure channel yet and we can unknowingly connect to a malicious third party which will provide with their own public key. And this is where **Certificate Authorities** (CAs) comes to the rescue. If Client trust CA -> and CA trusts Server -> then in theory Client can trust Server. This solution is called a chain of trust.

4. The operating system or browser is shipped with CAs public keys already stored on it. They are stored in client’s **Trusted Root Certificate Authority Store**. There are quite a lot of well-known and trustworthy CAs today. It may seem like a surprising number of keys to keep, but this is much easier than storing one public key for every website on the Internet.

5. Bringing it all together:

	Long before the handshake:

	- GitLab requests a certificate from CA
		- CA verifies that they are really talking to GitLab
		- GitLab sends CA their public key
		- CA uses their private key to digitally sign GitLab’s public key
		- CA gives GitLab the signed public key
		- This is now GitLab’s SSL Certificate (it also includes some additional information)

	Handshake:
	- **Hello**
		- Client connects to GitLab’s Server:
			- Client sends a _ClientHello_ message which contains the various cipher suites, maximum SSL version, compression methods that it supports, a random string that is used in subsequent computations
			- Server responds with a _ServerHello_ message which contains a decision based on the Client's preferences about cipher suite and version of SSL will be used. Server also sends its digital certificate and another random string. Optionally, if the server requires a digital certificate for client authentication, the server sends a _ClientCertificateRequest_ that includes a list of the types of certificates supported and acceptable CAs
		- Server sends to Client their SSL Certificate
	- **Certificate verification**:
		- Using the CA's public key in the root certificate store, Client verify CA’s signature on Server’s SSL Certificate
		- Optionally: server verifies the client's certificate
	- **Key exchange**:
		- Client generate a random secret key to be used for the main symmetric (which is faster than asymmetric) algorithm, and use GitLab’s public key from their certificate to encrypt it.
		- Client send the encrypted secret key to Server
		- GitLab decrypts it with their private key, and holds on to it.
	- **Finish**:
		- Client sends to the server a _Finish_ message, which is encrypted with the secret key, indicating that the client part of the handshake is complete.
		- Server sends to the client a _Finish_ message, which is encrypted with the secret key, indicating that the server part of the handshake is complete
		- Client and Server use the shared secret key to encrypt the traffic.

	![SSL/TLS handshake](https://www.ibm.com/support/knowledgecenter/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660a.gif)

### Browser's rendering engine

Now that the browser has the resources comprising the website (HTML, CSS, JavaScript, images, etc), it has to go through several steps to present the resources as a webpage.

The browser has a rendering engine that’s responsible for displaying the content. The rendering engine receives the content of the resources in small chunks. Then there’s an HTML parsing algorithm that tells the browser how to parse the resources.

Once parsed, it generates a tree structure of the DOM elements. DOM stands for Document Object Model and it is a convention for how to represent objects located in an HTML document. These objects — or “nodes” — of every document can be manipulated using scripting languages like JavaScript.

![DOM tree](https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/DOM-model.svg/440px-DOM-model.svg.png)

Once the DOM tree is built, the stylesheets are parsed to understand how to style each node. Using this information, the browser traverses down DOM nodes and calculates the CSS style, position, coordinates, etc for each node.

Once the browser has the DOM nodes and their styles, it is ready to present the webpage.

### TCP/IP

One thing worth noting is how information gets transmitted when you make a request for information. When you make a request, that information is broken up into many tiny chunks called packets. Each packet is tagged with a TCP header, which include the source and destination port numbers, and an IP header which includes the source and destination IP addresses to give it its identity. The packet is then transmitted through ethernet, WiFi or Cellular network and is allowed to travel on any route and take as many hops as it needs to get to the final destination.

(We don’t actually care how the packets get there — all that matters is that they get to the destination safe and sound!) Once the packets reach the destination, they are reassembled again and delivered as one piece.

So how do all the packets know how to get to their destination without getting lost?

The answer is TCP/IP.

TCP/IP is a two-part system, functioning as the Internet’s fundamental “control system”. IP stands for Internet Protocol; its job is to send and route packets to other computers using the IP headers (i.e. the IP addresses) on each packet. The second part, Transmission Control Protocol (TCP), is responsible for breaking the message or file into smaller packets, routing packets to the correct application on the destination computer using the TCP headers, resending the packets if they get lost on the way, and reassembling the packets in the correct order once they’ve reached the other end.


