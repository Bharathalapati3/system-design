
- Round Robin  - Stateless apps - Uniform distribution
- Least Connection - long live connections, file uploads
- IP Hash - Same client, same server
- Weighted Round Robin - Powerful server gets the most percentage of connections according to their weights


L4 - Network load balancing
L7 - Application Load balancing
   - nginx


Sticky session - When Load balance assign 1 session to the user always after initial request
                - When the server crash, the request goes to other server which is not aware of user session
                - best is to use redis which can share the session across multiple servers


  



