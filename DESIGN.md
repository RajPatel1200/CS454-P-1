# Project 1 - EC2 REST Service Design rationale

## Architecture
- EC2 instance serves as the server, and the local machine serves as the client
- The client sends the request via HTTP GET, and the server sends a JSON response back to client.

## Security 
- EC2 instance follows the principle of least privilege.
- Inbound rules in Security groups configured to limt SSH to user device's IP address
- Systemd runs as ubantu instead of root

## Technology Stack
- Node.js, Exprss.js and Morgan: A popular choice to handle REST APIs and integration with front-end JavaScript libraries
- Git and GitHub: For version control and for hositng git repository
- systemd: Service manager for the hosted API
  


