# Introduction #

It is well known that Aspect Oriented Programming can increase modularity of by allowing the separation of cross-cutting concerns, especially in web applications. Modern web applications have many concerns such as authentication, authorization, logging, caching, escape characters, and customization that clutters application code, and even HTTP API.

As HTTP API becomes increasing common nowadays, we can also see that popular HTTP API are usually bloated by such concerns. Hypershell is designed to avoid such bloat and allow script writers to design HTTP API that follows the Unix philosophy: make each HTTP API do one thing well. That is, scriptwriters should not be concerned with things such as authentication and should leave these concerns to _Hypershell Filters_.

# Details #

Hypershell filter allows web administrator to insert cross-cutting aspects onto any VRI path. When HTTP requests reach a VRI, the request is first processed and modified by before-filters before the Hypershell script is executed. After the Hypershell script returns a response, the response is processed by after-filters.

Hypershell filters are also Hypershell scripts. They accept and modify HTTP request and response as if it is a HTTP request to themselves. Therefore, it is possible to host Hypershell filter on different server from the target Hypershell script.


## Syntax ##

It is possible to add custom filter when issuing a Hypershell command. Such syntax is shown below:

```
[ before-filter-1, before-filter-2 [ command-1 | command-2 ] after-filter-1, after-filter-2 ]
```

When a Hypershell command contains filter, the command is surrounded by a double square bracket. A Hypershell filter can be inserted in between the two square bracket and separated by comma. Inside the double bracket one can pipe with more than one command before the after-filter takes place. A filtered Hypershell command must be enclosed by all four of the opening and closing brackets, even when there is no filter on one of the end.

The filter syntax allows programmers to add filters to Hypershell commands in a complex way that produce different results. Following are some examples of Hypershell commands with filter:

```
[ single-filter [ command ]]
[[ command ] after-filter ] | other command
[ before-filter [ command | [[inner-command]inner-filter] ]] 
```

## Common Usage on Hypershell filters ##

Hypershell filter can be used to implement many aspects that are essential to websites, such as:

  * Authentication
  * Authorization and Access Control
  * Cache Validation
  * Input Escape

for before-filter, and

  * Error Handling
  * Cache Update
  * Logging

for after-filter.

## Difference Between Filter and Ordinary Hypershell Command ##

You may wonder why is there a need to create Hypershell filter. Hypershell filter is designed to modify a target command, and a few features are added to allow Hypershell filter to perform things ordinary Hypershell command can't do.

Firstly, an ordinary Hypershell command has no knowledge of any preceding or proceeding Hypershell command, while Hypershell filter can inspect the entire HTTP message that is sent to or received from a Hypershell command. For example, it is particularly useful when Hypershell filter wants to perform authentication or cache validation based on the URL path of a request.

Secondly, Hypershell before-filter is able to short circuit the filter chain and return HTTP response on behalf of the target Hypershell command. While Hypershell after-filter is able to catch and receive the response message regardless of whether an error occured. These abilities allow Hypershell filters to alter the behavior and control flow of a series of Hypershell commands.

## Filter Example ##

To add permission feature to a blog website, developer can add an _access control_ filter in front of the blog post retrieval command:

```
[ check-read-blog-perm [ get-blog-post 123 ]]
```

From the above command, the `check-read-blog-perm` checks for the id parameter passed to `get-blog-post` and found the id to be 123. It can then query the database to check whether the logged in user has permission to read the blog. If the user has no permission, `check-read-blog-perm` returns a HTTP 401 Unauthorized response on behave of `get-blog-post`, and the actual command is skipped.

Throwing HTTP 401 directly to end user may not be user friendly. To enhance user experience, websites usually display an informative page about the error. We can add this feature easily using Hypershell filter without cluttering the original code:

```
[ check-read-blog-perm [ get-blog-post 123 ] log-unauthorized-access, unauthorized-redirect ]
```

We add the `log-unauthorized-access` to capture and log a HTTP 401 event. The HTTP 401 response is then passed unmodified to the `unauthorized-redirect`, which captures and replace the response with HTTP 303 See Other response which redirects user to a friendly error page.

With the example above, we can see how easy it is to alter the behavior of a simple `get-blog-post` command and add access control features to it.

### Result of using Pipe instead of Filter ###

On the above example, if Hypershell pipe is used instead of the filter syntax, we will get a different result:

```
check-read-blog-perm | get-blog-post 123 | log-unauthorized-access | unauthorized-redirect
```

Firstly, `check-read-blog-perm` would not be able to access the id parameter of 123 in `get-blog-post`, hence it is only able to check a general permission a user has instead of specific permission set on a blog post.

The after-filter `log-unauthorized-access` and `unauthorized-redirect` will be skipped when an actual HTTP 401 is raised. This is because HTTP error is treated as exception in Hypershell pipes and return immediately. As an ordinary Hypershell command, the filters also lose the ability to access the status code returned by the previous command.


## The message/http Content-Type ##

Hypershell filter is a specialized form of Hypershell command. A filter is also called through HTTP request and reply using HTTP response, which means a filter can also be located on different server from the target command.

To avoid ambiguity between the HTTP parameters for the filter and HTTP parameters for the target command, Hypershell filter always receive request and response in the message/http content type.

The message/http content type is actually an envelope that encapsulates a HTTP request or response message. With this content type, the whole request to the target command, including HTTP headers and request path, is be passed to the Hypershell before-filter as the content body. Same with response, the whole HTTP response, including headers and status code, is passed to the Hypershell after-filter as the content body.

The separation of message means that Hypershell filter may have its own sets of HTTP headers which is different from the target request headers. This also preserves the semantics of after-filter that the filter still receives a HTTP request, with a response encapsulated in the body, instead of receiving a HTTP response directly.


## Hypershell Filter Return Type ##

Hypershell filters also return message/http as the content type of their response to separate the meaning of HTTP status code.

Hypershell standard dictates that a Hypershell filter must only return either HTTP status 200 OK with a message/http body - to indicate a response replacement, or a HTTP status 304 Not Modified - to indicate it has not changed the target request/response. Any other status code is considered error and will generate HTTP status 500 on the calling script.

Referring back to the previous example, this means that if `check-read-blog-perm` wants to return a HTTP status 401, it cannot return the status code directly in the response. Instead, a HTTP status 200 OK is returned with the body containing a HTTP response envelope containing the HTTP status 401 code.