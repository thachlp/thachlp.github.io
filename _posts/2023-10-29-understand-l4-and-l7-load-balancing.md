---
layout: post
title: Understanding L4 and L7 Load Balancing
tags: [LoadBalancer]
comments: true
---

In the internet, where traffic constantly increases, 
load balancing is crucial in distributing network traffic across multiple servers. 
This ensures that no single server bears an excessive load, thereby improving the responsiveness and availability of applications. 
Two types of load balancing are commonly used: Layer 4 (L4) and Layer 7 (L7) load balancing. 

#### What is L4 Load Balancer
L4 load balancing operates at the transport layer (Layer 4) of the Open Systems Interconnection (OSI) model.
Layer 4 load balancing allows NGINX to distribute traffic based on network information such as IP address and TCP port.

The `stream { ... }` block is used for handling generic network stream traffic.
```
stream {
    upstream backend {
        server backend1.example.com:12345;
        server backend2.example.com:12345;
    }

    server {
        listen 12345;
        proxy_pass backend;
    }
}
```
#### What is L7 Load Balancer
L7 load balancing operates at the application layer (Layer 7) of the OSI model.
Layer 7 load balancing allows NGINX to decide how to distribute the requests based on the content of the HTTP header or the actual content of the message. 

The http `{ … }` is used for configuring HTTP (Layer 7) settings.
```
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```
### L4 vs L7 Load Balancing
1. **Complexity and Flexibility**: L7 load balancing is more complex and flexible than L4. 
L7 can make decisions based on the content of the message, allowing for more sophisticated routing rules. 
On the other hand, L4 load balancing is simpler and faster, as it only needs to look at the IP addresses and ports.

2. **Performance**: L4 load balancing generally performs better than L7 because it deals with less information, 
resulting in faster processing times. However, L7 load balancing, while slower, offers a more granular level of control, 
enabling advanced features like SSL offloading, cookie-based session persistence, and content-based routing.

3. **Use Cases**: L4 load balancing is typically used when you need simple, fast, 
and efficient distribution of network traffic. It's ideal for TCP/UDP traffic, VPNs, and databases. 
L7 load balancing is used when you need to make routing decisions based on the application content. 
It's ideal for HTTP/HTTPS traffic, web applications, and microservices architectures.

For very basic [Nginx take note](https://github.com/thachlp/second-brain/tree/main/nginx)
