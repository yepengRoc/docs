原文地址：

[http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.dre.vanderbilt.edu%2F~schmidt%2FPDF%2Freactor-siemens.pdf)

 



The main program creates a Logging Acceptor, whose
constructor initializes it with the port number of the logging server. The program then enters its main event-loop.
Subsequently, the Initiation Dispatcher Singleton
uses the select event demultiplexing system call to synchronously wait for connection requests and logging records to arrive from clients.
The following interaction diagram illustrates the collabo-
ration between the objects participating in the loggingserver
example:  

Once the Initiation Dispatcher object is initialized, it becomes the primary focus of the control flow within
the logging server. All subsequent activity is triggered by
hookmethodsontheLogging AcceptorandLogging
Handler objects registered with, and controlled by, the
Initiation Dispatcher.  
When a connection request arrives on the network
connection, the Initiation Dispatcher calls back
the Logging Acceptor, which accepts the network
connection and creates a Logging Handler. This
Logging Handlerthenregisterswith theInitiation
Dispatcher for READ events. Thus, when a client sends
a logging record, the Initiation Dispatcher calls
back to the client’s Logging Handler to process the incoming record from that client connection in the logging
server’s single thread of control.  
10 Known Uses  
The Reactor pattern has been used in many object-oriented
frameworks, including the following:
? InterViews: The Reactor pattern is implemented by
the InterViews [10] window system distribution, where
it is known as the Dispatcher. The InterViews
Dispatcher is used to define an application’s main event
loopandtomanageconnectionstooneormorephysicalGUI
displays.  
? ACE Framework: The ACE framework [11] uses the
Reactor pattern as its central event demultiplexer and dispatcher.  
The Reactor pattern has been used in many commercial
projects, including:  
? CORBA ORBs: The ORB Core layer in many single-
threaded implementationsof CORBA [12] (such as VisiBroker, Orbix, and TAO [13]) use the Reactor pattern to demul-
tiplex and dispatch ORB requests to servants.  
? Ericsson EOS Call Center Management System: This
system uses the Reactor pattern to manage events routed by
Event Servers [14] between PBXs and supervisors in a Call
Center Management system.  
? ProjectSpectrum: Thehigh-speedmedicalimagetrans-
fer subsystem of project Spectrum [15] uses the Reactor pat-
tern in a medical imaging system.  
11 Consequences  
11.1 Benefits  
The Reactor pattern offers the following benefits:  

Separation of concerns: The Reactor pattern decouples application-independent demultiplexing and dispatch-
ing mechanisms from application-specific hook method
functionality. The application-independent mechanisms become reusable components that know how to demultiplex
events and dispatch the appropriate hook methods defined
by Event Handlers. In contrast, the application-specific
functionality in a hook method knows how to perform a par-
ticular type of service.
Improve modularity, reusability, and configurability of
event-driven applications: The pattern decouples application functionality into separate classes. For instance, there
are two separate classes in the logging server: one for establishing connections and another for receiving and processing logging records. This decoupling enables the reuse
of the connection establishment class for different types of
connection-oriented services (such as file transfer, remote
login, and video-on-demand). Therefore, modifying or extending the functionality of the logging server only affects
the implementation of the logging handler class.
Improves application portability: The Initiation
Dispatcher’s interface can be reused independently of
the OS system calls that perform event demultiplexing.
These system calls detect and report the occurrence of one
or more events that may occur simultaneously on multiple sources of events. Common sources of events may include I/O handles, timers, and synchronization objects. On
UNIX platforms, the event demultiplexing system calls are
called select and poll [1]. In the Win32 API [16], the
WaitForMultipleObjectssystem call performsevent
demultiplexing.  
Provides coarse-grained concurrency control: The Reactor pattern serializes the invocation of event handlers at
the level of event demultiplexing and dispatching within
a process or thread. Serialization at the Initiation
Dispatcher level often eliminates the need for more complicated synchronization or locking within an application
process.
11.2 Liabilities
The Reactor pattern has the following liabilities:
Restricted applicability: The Reactor pattern can only be
applied efficiently if the OS supports Handles. It is possible to emulate the semantics of the Reactor pattern using
multiple threads within the Initiation Dispatcher,
e.g. one threadfor eachHandle. Wheneverthere are events
availableonahandle,itsassociatedthreadwill readtheevent
and place it on a queue that is processed sequentially by the
initiation dispatcher. However, this design is typically very
inefficientsince it serializes all Event Handlers, thereby
increasing synchronization and context switching overhead
without enhancing parallelism.

Non-preemptive: In a single-threaded application process, Event Handlers are not preempted while they are
executing. This implies that an Event Handler should
not perform blocking I/O on an individual Handle since
this will block the entire process and impede the responsiveness for clients connected to other Handles. Therefore,forlong-durationoperations,suchastransferringmulti-
megabyte medical images [15], the Active Object pattern
[17] may be more effective. An Active Object uses multi-threading or multi-processing to complete its tasks in parallel
with the Initiation Dispatcher’s main event-loop.
Hardto debug: Applicationswritten with theReactor pattern can be hard to debug since the inverted flow of control oscillates between the framework infrastructure and the
method callbacks on application-specific handlers. This increases the difficulty of “single-stepping” through the runtime behavior of a framework within a debugger since application developers may not understand or have access to the
frameworkcode. Thisis similar to the problemsencountered
trying to debug a compiler lexical analyzer and parser written with LEX and YACC. In these applications, debugging
is straightforward when the thread of control is within the
user-defined action routines. Once the thread of control returns to the generated Deterministic Finite Automata (DFA)
skeleton, however, it is hard to follow the program logic.
12 See Also
The Reactor pattern is related to the Observer pattern [5],
where all dependents are informed when a single subject
changes. In the Reactor pattern, a single handler is informed
when an event of interest to the handler occurs on a source
of events. The Reactor pattern is generally used to demultiplex events from multiple sources to their associated event
handlers, whereas an Observer is often associated with only
a single source of events.
The Reactor pattern is related to the Chain of Responsibility (CoR) pattern [5], where a request is delegated to the responsible service provider. The Reactor pattern differs from
the CoR pattern since the Reactor associates a specific Event
Handler with a particular source of events, whereas the CoR
pattern searches the chain to locate the first matching Event
Handler.
The Reactorpattern canbe considereda synchronous variant of the asynchronous Proactor pattern [18]. The Proactor supports the demultiplexing and dispatching of multiple
event handlers that are triggered by the completion of asynchronous events. In contrast, the Reactor pattern is responsible for demultiplexing and dispatching of multiple event
handlers that are triggered when it is possible to initiate an
operation synchronously without blocking.
The Active Object pattern [17] decouples method execution from method invocation to simplify synchronized access
to a shared resource by methods invoked in different threads
of control. The Reactor pattern is often used in place of the

Active Object patternwhenthreadsare not availableor when
the overhead and complexity of threading is undesirable.
An implementation of the Reactor pattern provides a Facade [5] for event demultiplexing. A Facade is an interface
that shields applications from complex object relationships
within a subsystem.

