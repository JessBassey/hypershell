## Introduction ##

Hypershell is an extension to the Hypertext Transfer Protocol to allow web applications be developed web in a shell script fashion. This is done by creating a _service oriented architecture_ for web applications that is based on plain old HTTP subrequests. (No, Hypershell is not a way for you to access your Unix shell through HTTP)

Hypershell is implemented with a collection of tools, including a HTTP extension protocol, a DSL programming language, and an [application server](Tevatron.md).

Hypershell code repository is now available at [GitHub](https://github.com/hypershell/hypershell).

A presentation slide about the overview of Hypershell is available [here](http://docs.google.com/a/hypershell.org/present/view?id=dzqvrxf_0wr6srr94).

## Philosophy ##

Hypershell is heavily inspired from the Unix Shell. It treats every resource located at a HTTP URI as a Hypershell command. A Hypershell command is similar to a Unix command but differs in several ways: While traditional Unix commands accept input from STDIN, and print output to STDOUT, and STDERR, Hypershell commands accept input from a HTTP request and return output in the form of HTTP response.

Unlike Unix shell command which create new processes every time the command is executed, a Hypershell command is a persistent servlet that runs independently from other servlets and is reused for every subsequent HTTP requests.

A Hypershell command can be written in any programming language. Hypershell command shares the Unix philosophy of Make each program do one thing well. Therefore, each Hypershell command focuses on one specialize task delegates other tasks to other commands.

## A Simple Example ##

Supposed that `get-blog-post` is a Hypershell command that returns a blog post content in JSON format and `render-html-blog` is another Hypershell command that accepts a JSON blog post and render it in HTML, the two commands can be combined and become

```
get-blog-post 3062 | template-render templates/view-blog.html
```

The `get-blog-post` command can be decomposed into several hypershell commands that fetch a blogpost from a database. For example, using the Model pattern in MVC we can also directly render an ajax enhanced blog using the following Hypershell command:

```
model/gen-sql-select model-def/blogpost | db/mydb/sql-query id=123 | render-ajax-blog
```

## Peer to Peer HTTP ##

Because a Hypershell command almost always issue a chain of other Hypershell commands, we can describe Hypershell as Peer to Peer HTTP. Hypershell changes the original perspective of HTTP that all HTTP requests must come from external clients, and a web application is a giant monolithic collection of libraries that serve these external clients. Instead from Hypershell’s perspective, a web application is expected to receive HTTP requests from both external clients and other internal peer web applications, and the application is encouraged to issue HTTP requests to other web applications.

In peer to peer (P2P) HTTP design, every web application as both HTTP client and server. This is different from the design of current web servers, which does not provide optimized facilities for internal applications to issue HTTP requests. A HTTP request coming from a peer server/application should also be interpreted differently from HTTP requests coming from actual end clients. For instance, a peer HTTP request should include the IP address and authenticated credentials of the end user instead of the requesting server.

As a result, current web servers have to be redesigned to be Hypershell/P2P HTTP friendly. A set of HTTP extension protocol is also needed to optimize the communication between trusted peer servers. Tevatron is a new application server designed for hosting Hypershell applications. I will refer to Tevatron frequently when discussing about the implementation of web server for Hypershell.

# Development Status #

This project is still under early stage of development. No code is written yet as there are architectural decisions to make and I'm still lack of knowledge and experience. I'll be grateful if people with experience can assist me on writing the web server and protocol specification.

An RFC will be written to propose the extension of HTTP protocol to support Hypershell. The Tevatron server will be written in C, and provide standard API for different programming language implementations to interact with the server. New shell-like DSL will be created and research will be done on how to constraint existing languages to run as Hypershell scripts in a secure, portable, and scalable way.

Check out our [wiki](http://code.google.com/p/hypershell/w/list) to learn more about Hypershell. Please join our [Google Group](http://groups.google.com/group/hypershell) to discuss and take part in the implementation of Hypershell.