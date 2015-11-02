# Introduction #

Web application programming is very challenging due to the vast number of disciplines involved. Development of contemporary web applications usually involves compromises between different features because there is no general solution that can solve all the problems at the same time.

# Desired Features in Web Programming #

The desired features when writing web applications are listed below. We discuss the challenge of implementing such features and how Hypershell can help to solve the problem and implement the features.

## Simplicity ##

Web developers want to write simple code. They do not want to care about other features and components in a system, write boilerplate code to implement other desired features and instead expect the system to have the features automatically. Web developers want to write code that does what they want, i.e. the application code proper, and does so without having to understand other concerns and the inner working of the system.

Making program looks simple seems easy, however maintaining simplicity while retaining other features is hard. For example, to write modular code web programmers usually have to understand the big picture of a system or framework, how to interface and exchange parameters with the system, and code the program in a very specific way.

Hypershell allows web developers to code in ways that are simple and familiar to them, by following the Unix shell philosophy: make each program do one thing well. By delegating the actual tasks to subrequests, developers can focus on solving the problem without having to be knowledgeable on any other part of the system. Hypershell [filters](Hypershell_Filter.md) also helps to free developers of other cross-cutting concerns.

## Modularity ##

Web developers want to write components that are modular. They want to write the code once and use it everywhere they want. While it seems easy to achieve modularity in simple programming, modularity has not actually improved much in web development.

Firstly, as web applications are typically divided into several layers, with one example being the MVC design pattern, a component's code is scattered around at least these 3 layers. Making the Model modular is hard as it involves database schema; Making the View modular requires abstraction over raw HTML to present a unified layout along with other components; Making the Controller modular is easier, however the way user can call for data is fixed and the controller is required to handle too many different operations such as in RESTful controller. Therefore, not only code scattering harms modularity, it exposes too many concerns to web developers and draws their attention away from actual coding.

Modularity also comes at the cost of code coupling, and that makes modularity of limited usefulness to external codes that are not coupled in the same way. For example, modules written in a language cannot be shared easily with components written in another language. Even modules written in different frameworks are usually difficult to communicate with each other. Frequently this forces developers to either be limited to a particular framework, or reinvent the wheel all over again.

## Extensibility ##

## Security ##

## Scalability ##

## Availability ##

## Portability ##

## Native Processing ##

## Code Decoupling ##
### Language Decoupling ###
### Framework Decoupling ###
### Library Decoupling ###
### Aspects Decoupling ###