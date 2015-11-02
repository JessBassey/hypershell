# Introduction #

Because a Hypershell command serves a web application through HTTP request, it is constrained by the limitations of HTTP as well. However the constraints given by HTTP is actually a good thing, because it forces developers to write code in a scalable and platform independent way. By following these constraints, one can eventually appreciate the beauty of HTTP and become a better programmer.


## Statelessness ##

The first important constraint is that HTTP is designed to be _stateless_. If a Hypershell command is completely stateless, it returns the same result every time when the same parameter is given. Statelessness is very important for website performance because the application server can cache the result of a Hypershell command and use the cache to serve subsequent requests with the same parameter.

A stateless Hypershell command also closely resembles a _pure function_ in functional programming and enjoy the same benefits. Other than enjoying automatic _memoization_ in the form of cache, statelessness also allow many copies of a Hypershell command to run concurrently. This allows a Hypershell web application to scale out of the box by simply adding more threads and servers for resource intensive Hypershell commands.

Unfortunately there are situations where it is necessary to carry states such as user credentials across Hypershell application. A Hypershell command is _stateful_ if it reads from parameters other than GET and POST content, such as the Cookie and X-Forwarded-For HTTP headers. These headers are usually almost unique for every HTTP requests, and are rare enough to not worth caching. Therefore a stateful Hypershell command is either not cacheable or requires strong cache validation, giving potential performance overhead to the server.

Developers should avoid writing stateful whenever possible, and improve statelessness by using stateful commands to delegate the actual task to other stateless commands.

## Pass By Value ##

As Hypershell commands communicate by issuing HTTP requests, from a programming perspective this means that the parameters are _pass by value_. Since a copy of the variable is passed instead of reference pointer, a Hypershell command is side-effect free and cannot change the state of the original copy of value.

While it is possible for a Hypershell command to store internal state, it cannot share the state with other Hypershell commands. One can simulate global variable in Hypershell by querying a command that contains a stateful counter or a database, however the value that is returned by this command is only a copy of its internal state. To verify if the internal state is updated, one must continuously query the command to get the update.

The pass by value nature disallows Hypershell commands to directly share state with each other, thereby avoid the common problems of having shared states in concurrent programs. Although pass by value can be inefficient and create performance impact, application server such as Hyggs uses copy on write technique when HTTP requests are called internally. Nevertheless, we believe that the trade off is worth for writing modular Hypershell commands that call by value, compared to writing monolithic application that communicate internally using call by reference.

## Short Lived ##

HTTP session is _short lived_ and time out quickly. This characteristic has been commonly complained by Ajax developers who wish to build long lived Ajax applications. However there are benefits of making process cycles of Hypershell command stateless and short lived. For instance, administrator can transparently move a Hypershell command to another server at any time, and stop the old command soon after the current requests are responded. Short lived HTTP connection together with statelessness also makes hot update possible and reduces the administrative overhead when scaling website.