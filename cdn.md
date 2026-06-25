# Content-Delivery Network

A CDN is a network of edge servers distributed around the world that cache your content close to your users — solving the fundamental problem of physics: light takes time to travel.

- static files can be cached at the cdn server
- if cache miss, it calls the origin server
- Two strategies
   - Cache pull - first request from user will be served from origin and then the reponse is cached and served to all other users
   - Cache push - origin server pushes the new content to cdn server periodically and invalidates the cache.
 
- the origin sets s-maxage to determine how long the content should be cached in cdn
 
- Not all the user requests will be served from CDN as it servers only cached content.
 
- New mechanism - modern CDN server
     - This mechanism does more than cache, and static content handling. It also handles the dynamic changing data like user auth,personalization, state managmeent
 

 
Products: Cloudflare Workers, Lambda@Edge, and Fastly Compute
