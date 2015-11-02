# Introduction #

The current HTTP protocol do not cover well on circumstances when subrequests are issued within an internal website. In order for Hypershell to work well with HTTP, a few protocol extension is required mainly to standardize the convention of handling responses from subrequests.

# Participants in HTTP #

From section 1.3 and 1.4 of [RFC2616](http://tools.ietf.org/html/rfc2616), HTTP/1.1 defines a number of participants that take part in a HTTP request/response:
  * Client
  * Server
  * Proxy
  * Gateway
  * Tunnel

The protocol assumes a simple picture of interaction: a client initiates a request, the server processes the request and return a response. The server side processing is usually a single monolithic step, with little subsequent interactions.

## Breaking the Assumption ##

Hypershell breaks the assumption of HTTP by introducing subrequest as the basis of server side processing. This changes the basic nature of HTTP from being a client-server protocol to a peer to peer protocol.

Breaking the assumption may introduce consequences to current web server and applications.   Current web servers and applications typically assume that a HTTP request comes from an external client. When an internal server component issues a subrequest, the server/application cannot differentiate the subrequest from client request, and may produce unexpected behavior that may compromise integrity and security of the application.

On the requester side, there is also no facility to pass on the information from an original request to the subrequest. For example, context sensitive information such as IP address, user agent, and cookies of a subrequest is different from the original request.

The loss of information makes it very hard for the recipient to deduce the identity of the user, and functionality such as authentication may break down.

## Participants in Hypershell ##

Hypershell introduces new participants and replaces some old participants in the HTTP interaction diagram:

  * End Client
  * Peer server
    * Requester peer
    * Recipient peer
  * Gateway
    * Inbound Gateway
    * Outbound Gateway
  * Proxy
  * Tunnel

Firstly, two types of peers is introduced to replace server, the requester peer and the recipient peer, which are logical entities reside on one or more peer servers. Secondly, the gateway is divided into two participants: the inbound gateway (aka reverse proxy), and the outbound gateway (aka forward proxy).

In Hypershell's interaction diagram, an inbound gateway MUST exist to handle HTTP requests coming from external clients. The inbound gateway then acts as a requester peer, filters the request and reissue it as a subrequest to a peer server. From the peer server's point of view, all request is always subrequest and is always coming a trusted peer. Therefore it is important to filter all untrusted HTTP requests using the inbound gateway, even when there is only one peer server.

Based on this requirement and assumption of a peer only receives subrequest from trusted source, we can build more flexible rules into Hypershell's HTTP protocol, which is different from the current HTTP model that assumes all HTTP requests come from untrusted source.

# Trusted HTTP Headers #

Because a peer recipient always receive requests only from trusted peers, we can incorporate trusted HTTP headers into the Hypershell protocol. The headers are useful for a Hypershell peer to preserve request information when issuing subrequests.

First of all, the famous X-Forwarded-For is the standard way to retrieve IP address information in Hypershell. In fact, Hypershell peer SHOULD NOT retrieve IP address based on the requester's IP address.

Hypershell also extends the X-Forwarded family header to forward other information of the original request. Some tentative examples are the X-Forwarded-User-Agent to forward User-Agent, and the X-Forwarded-Cookie to forward cookies.

Hypershell also introduces new trusted HTTP headers to make ease of common design patterns in web application.

### Session-Id ###
The Session-Id header is used to identify the user identity of an authenticated session that can be derived from cookies using Hypershell filters.

### Content-Escape ###
The Content-Escape header defines the escape methods that have already been applied to the request content. This is useful to avoid repeated escape of the request content done by multiple Hypershell filters.

This header also decouples the escape operation from the actual escape algorithm. When the Content-Escape header exists, the peer SHOULD trust the result and proceed with processing the request. The peer MAY re-validate the request body, but MUST NOT attempt to re-escape the request in case the the escaping is found to be invalid. Instead, a HTTP 500 error must be thrown to indicate the failure of content escaping.

The peer MAY assert its expectation of the request content to be escaped using a specific escape method. If such escape method is not found, the peer MUST throw a HTTP 500 error to indicate the expectation failure. The peer SHOULD NOT attempt to escape the request body using its own method, as doing so will increase the complexity and reduce the modularity of the code.

Content-Escape provides the following standard methods of escaping:
  * html - the content is HTML escaped and is safe to be displayed as plain text.
  * url - the content is url encoded and is safe to be used inside a HTTP address.
  * javascript - the content may contain HTML but javascript functions are escaped. There should be no Javascript XSS vulnerability associated with this content.

### Script-Owner ###

This is Hypershell's loose equivalent to the file owner in the Unix filesystem. An owner may be assigned to a hypershell script to fine grain the access control between Hypershell scripts. A recipient peer MAY use this header to check whether the indicated owner has permission to call the target Hypershell command.

The Script-Owner is a simple access control mechanism to control between code written by different authors with no authentication built in. An application SHOULD also check for the permission of the authenticated user using the Session-Id header.

# HTTP Status Code in Hypeshell #

Because the request a recipient peer receives may also be a subrequest initiated by other peer, HTTP status other than 200 OK needs to be reinterpreted and the way to handle the status code from the requester peer should be standardized.

Non 200 status code usually indicate an exception on the requester peer. If the requester peer do not action upon the status code, the error of a sub-subrequest should either be passed down and returned as the response of the subrequest, or a 500 Internal Server should be raised to indicate the failure of handling such status code. The error status code acts like exception in object oriented programming that it aborts following operations of the recipient and return immediately to the requester.

However, not all status code should be passed down unchanged along the subrequest chain. For example, passing down a 3XX redirection to external site may expose the website to phishing attack, and such vulnerability must be avoided. Instead, the default behavior is to re-raise the error as status 500.

Following is the reinterpretation of HTTP status codes and the default behavior to handle them:

## 200 OK ##
Indicates successful return and the requester may proceed on processing the content.

## 206 Partial Content ##
Hypershell uses this status to optimize stream handling in subrequests. Further explanation is available [here](Hypershell_Stream_Processing.md).

## Other 2XX Status ##
Considered the same as 200 OK. If no content is presented in the response, the content is treated as an empty string.

## 300 Multiple Choices ##
Treated as a normal return but may be acted differently from 200 OK.

## 301 Moved Permanently ##
If the location url has the same domain as the requesting domain, the requester peer MAY re-issue the request to the given url. Otherwise, the requester peer must decide what to do with the status or status 500 will be raised.

## 302 Found ##
This status is depreciated and should not be used in Hypershell. In case such status is received, the requester peer must decide for an action or status 500 will be raised.

## 303 See Other ##
This is the default behavior of passing a redirection down to the end client. The location URL must belong to the same domain as the website. Otherwise the requester must pass down the redirection explicitly or status 500 will be raised. If the url is on the same domain, the remaining operation on the requester peer is aborted and this status is returned instead.

## 304 Not Modified ##
The requester peer may use the cached copy of response obtained previously. For Hypershell [filter](Hypershell_Filter.md), this indicates the request/response hasn't been changed and the original request/response should be returned instead.

## 307 Temporary Redirect ##
Same as status 301, the requester peer should redo the request on the given url. However different from 301, the requester peer must explicitly issue the re-request or error 500 will be raised.

## 400-403, 405-407 Status ##
These status indicate that the end client is not allowed to perform the request, thus by default the status is passed down to the end client by default.

## 404 Not Found ##
This status has ambiguous meaning in Hypershell and should be used carefully. The original interpretation may mean that a particular resource is not found or the server/request handler is not found. To resolve the ambiguity, this status always mean the former interpretation, i.e. connection to the recipient peer is successful, but the handler can't found any associated resource. For connection failure or not found request handler, status 410 is used instead.

The requester peer may always assume that the Hypershell command exists but fail to find the resource when expecting status 404. Hence the requester peer can make the correct decision when handling this error. However status 404 must be handled explicitly or error 500 will be raised.

## 410 Gone ##
In Hypershell, this status is used instead to indicate connection failure and failure to find a request handler. This usually indicates a more serious failure than status 404 and the requester peer SHOULD abort remaining operations. The default way to handle such error is to reraise the error as status 500.

## 500 Internal Server Error ##
This is the most common used status to indicate server side error. The error is always passed down if not handled. A trace may be included with this status to assist on debugging.

## 5XX Server Side Errors ##
All of the 5XX statuses MUST be handled explicitly or the error will be reraised as error 500.






# References #
http://tools.ietf.org/html/rfc2616