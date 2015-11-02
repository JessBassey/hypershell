# Introduction #

The architecture of Tevatron is more complicated and different from traditional web servers, due to the interactive nature of Hypershell scripts. Tevatron is also designed from scratch to be of high performance, allowing huge amount of user space "green threads" to be run concurrently.

The design of Tevatron makes frequent reference to many modern high performance architecture such as Nginx, Stackless Python, Erlang, and GHC.


## Thread Based Control Flow ##

Tevatron processes each HTTP request in a separate user space thread, or properly called continuation. In other words, for every incoming HTTP request or a HTTP subrequest issued by internal Hypershell scripts, an entirely new user thread is created. (!) This control flow is different from [event/actor based concurrency](Event_Based_Concurrency.md), which is implemented on other high performance systems such as Erlang.

This approach means that huge amount of user threads will be created, and there must be a way for Tevatron to schedule these threads efficiently. To further complicate the problem, each thread in turn can spawn new threads when a Hypershell script issues subrequest, and the parent thread has to be suspended to wait for response.

Tevatron uses various techniques to handle the creation and scheduling of these user threads, which are explained below.


## Continuation ##

Thread can mean a lot in programming. Firstly there are kernel threads, which are heavy weight threads that created and scheduled by the kernel as if they are processes. Secondly, there are user space threads, also known as green threads, which are actually execution contexts that are idle most of the time and awaiting to be run by worker threads. Worker threads, on the other hand, are actually kernel threads that fetch user threads from the user space scheduler and run the user thread.

To avoid confusion on the ambiguous meaning of thread, we use the term _continuation_ when we refer to the user thread.

The user thread can also more loosely defined as execution context when viewed from the C programming perspective. The execution context is actually a separate C stack that a C program jumps to. As the execution context is closely related to the C stack, it is inefficient and cannot be applied easily to high level languages that also use the C stack for other purposes.

Continuation provides high level construct to the program control flow, and can be used across different languages as long as the languages supports continuation. It is also more accurate to refer to user thread as continuation, since continuation provides better semantics to the program and can give us a better conceptual picture on how Tevatron works.

## Language Requirement ##

There are several requirements for high level scripting languages to participate in Tevatron as Hypershell scripts. First of all the language implementation has to hook into Tevatron's HTTP subrequest API to allow scripts to issue subrequest.

Other than that, the implementation should not perform synchronous I/O directly to the kernel, as doing that will block Tevatron's worker thread. All I/O must be done asynchronously through Tevatron's AIO API and the implementation should return a continuation if it wishes to wait for the I/O result.

Hypershell scripts written in the given language must be reentrant and there should be no shared global variables. The internal state of the program is not guaranteed to persist as Tevatron may move the code to other servers. Hypershell scripts are expected to be functional and always return the same response if the request parameters and subrequest results are the same. Language implementation should enforce such behavior or caching on the script must be disabled and it may impact Tevatron's performance.

Lastly and most importantly, the language implementation must support continuation. During the event of awaiting AIO and subrequests, the language implementation should return a continuation to allow Tevatron's worker thread to process other requests. While the bad news is that the current major language implementations do not support continuation, the good news is that many alternative language implementations such as Stackless Python are under active development. Hence, Tevatron is designed to be incompatible with the old languages that do no support continuation, but we will work hard to ensure the alternative implementations are runnable on Tevatron without much overhead.

## Architecture Overview ##