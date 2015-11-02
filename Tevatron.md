# Introduction #

Tevatron is the codename for a high performance web application server that will implement Hypershell. The name Tevatron comes from the [Tevatron Particle Accelerator](http://en.wikipedia.org/wiki/Tevatron).


## A Different Server Architecture ##
A new web server has to be written to implement Hypershell due to significant difference in architecture requirements compared to traditional web applications.

### Internal HTTP Request ###

The P2P HTTP nature of Hypershell applications will generate a lot of internal activities between servlets. Traditional web servers such as Apache are not designed to handle such internal HTTP requests.

To reduce performance overhead of interaction between servlets in the same machine, virtual HTTP connection is managed by Tevatron where request structures are directly passed by pointer to the target servlet. Tevatron also implements copy-on-write mechanism on request structures so that the target servlet cannot change any shared value that can alter the semantics of pass-by-value in HTTP requests.

From the recipient servlet's point of view, internal request is indistinguishable from remote HTTP requests.

### Virtual URI System ###

Tevatron provides further HTTP request abstraction by providing a virtual URI system. The system dynamically maps a local URI to a real HTTP URL or to other servlets on the same server. The virtual URI system can also map local URIs using different protocols such as HTTPS or Tor, or map to a file located on local filesystem, or even a Unix shell command.

### Server Side Caching ###

With HTTP request comes web caching. Caching has become increasingly important on server side, and Hypershell applications benefit from automated caching mechanism based on HTTP.

However current web servers have very little support for server side caching. A different HTTP caching system has to be build for Hypershell, as it has different caching requirements from a browser or proxy cache.

High performance server side caching would require all cache to be stored in memory, and on the same memory space as the server process. The cache is managed by Tevatron and is shared among all local servlets. Tevatron also cache the processed data structures of HTTP responses to be reused directly.

### Filter System ###

Hypershell provides Aspect Oriented Programming by implementing a filter system on Tevatron's virtual URI system. A Hypershell filter command will be called before or after HTTP request on the specified URI to alter the behavior of the original command.

Tevatron allows an administrator to insert Hypershell filter to any URI on any time. This approach is different from the traditional web server where resources on URI are fixed on server startup and are configured using complex configuration files.


### Multi Threading and Continuation ###

Tevatron have the opportunity to take a different approach on multi threading from traditional web servers thanks to the nature of Hypershell. When a servlet issues HTTP request to other remote server, its execution context can be suspended and stored in a continuation-like structure. The thread running the servlet can then continue to process a new request while waiting for the remote response.

Responses returned to Tevatron are then stored in a thread-safe queue, where free threads fetch the response and proceed to continuation.