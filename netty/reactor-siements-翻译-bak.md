
9 Implementation  
This section describes how to implement the Reactor pattern
in C++. The implementation described below is influenced
by the reusable components provided in the ACE communication software framework [2].  
9.1 Select the Synchronous Event Demultiplexer Mechanism  
The Initiation Dispatcher uses a Synchronous
Event Demultiplexerto wait synchronouslyuntil oneor more events occur. This is commonly implemented us-ing an OS event demultiplexing system call like select.
The select call indicates which Handle(s) are ready to
perform I/O operations without blocking the OS process in
which the application-specific service handlers reside. In
general, the Synchronous Event Demultiplexer
is based upon existing OS mechanisms, rather than developed by implementers of the Reactor pattern.  
9.2 Develop an Initiation Dispatcher  
The following are the steps necessary to develop the
Initiation Dispatcher:
Implement the Event Handler table: A Initiation
Dispatcher maintains a table of Concrete Event
Handlers. Therefore, the Initiation Dispatcher
provides methods to register and remove the handlers from
this table at run-time. This table can be implemented in various ways, e.g., using hashing, linear search, or direct indexing if handles are represented as a contiguous range of small
integral values.   
Implement the event loop entry point: The entry point
into the event loop of the Initiation Dispatcher
should be provided by a handle events method. This
method controls the Handle demultiplexing provided by
the Synchronous Event Demultiplexer, as well as
performing Event Handler dispatching. Often, the main
event loop of the entire applicationis controlledby this entry point.  
When events occur, the Initiation Dispatcher
returns from the synchronous event demultiplexing call
and “reacts” by dispatching the Event Handler’s
handle event hook method for each handle that is
“ready.” This hook method executes user-defined code and
returns control to the Initiation Dispatcher when it
completes.  
The following C++ class illustrates the core methods on
the Initiation Dispatcher’s public interface:

```c++
 enum Event_Type
   // = TITLE
   // Types of events handled by the
   // Initiation_Dispatcher.
   //
   // = DESCRIPTION
   // These values are powers of two so
   // their bits can be efficiently ‘‘or’d’’
   // together to form composite values.
   {
   ACCEPT_EVENT = 01,
   READ_EVENT = 02,
   WRITE_EVENT = 04,
   TIMEOUT_EVENT = 010,
   SIGNAL_EVENT = 020,
   CLOSE_EVENT = 040
   };
   class Initiation_Dispatcher
   // = TITLE
   // Demultiplex and dispatch Event_Handlers
   // in response to client requests.

{
public:
// Register an Event_Handler of a particular
// Event_Type (e.g., READ_EVENT, ACCEPT_EVENT,
// etc.).
int register_handler (Event_Handler *eh,
Event_Type et);
// Remove an Event_Handler of a particular
// Event_Type.
int remove_handler (Event_Handler *eh,
Event_Type et);
// Entry point into the reactive event loop.
int handle_events (Time_Value *timeout = 0);
};
```

Implement the necessary synchronization mechanisms:
If the Reactor pattern is used in an application with only one
thread of control it is possible to eliminate all synchronization. In this case, the Initiation Dispatcher serializes the Event Handler handle event hooks within
the application’s process.
However, the Initiation Dispatcher can also
serve as a central event dispatcher in multi-threaded applications. In this case, critical sections within the Initiation
Dispatcher must be serialized to prevent race conditions
when modifying or activating shared state variables (such as
the table holdingthe Event Handlers). A common technique for preventing race conditions uses mutual exclusion
mechanisms like semaphores or mutex variables.
To prevent self-deadlock, mutual exclusion mechanisms
can use recursive locks [4]. Recursive locks hold prevent
deadlock when locks are held by the same thread across
Event Handler hook methods within the Initiation
Dispatcher. A recursive lock may be re-acquired
by the thread that owns the lock without blocking the
thread. This property is important since the Reactor’s
handle eventsmethodcallsbackonapplication-specific
Event Handlers. Application hook method code may
subsequently re-enter the Initiation Dispatcher
via its register handler and remove handler
methods.  

9.3 Determine the Type of the Dispatching
Target  
Two different types of Event Handlers can be as-
sociated with a Handle to serve as the target of an
Initiation Dispatcher’s dispatching logic. Implementations of the Reactor pattern can choose either one or
both of the following dispatching alternatives:
Event Handler objects: A common way to associate an
Event Handler with a Handle is to make the Event
Handleranobject. Fori nstance,the Reactor pattern implementation shown in Section 7 registers Event Handler
subclass objects with an Initiation Dispatcher.
Using an object as the dispatchingtarget makes it convenient
to subclass Event Handlersin orderto reuse and extend

existing components. In addition, objects integrate the state
and methods of a service into a single component.
Event Handler functions: Another way to associate an
Event Handler with a Handle is to register a function
with the Initiation Dispatcher. Using functions as
the dispatching target makes it convenient to register callbacks without having to define a new class that inherits from Event Handler.  
The Adapter pattern [5] be employed to support both
objects and functions simultaneously. For instance, an
adapter could be defined using an event handler object that
holds a pointer to an event handler function. When the
handle event method was invoked on the event handler
adapter object, it could automatically forward the call to the
event handler function that it holds.
9.4 Define the Event Handling Interface
Assuming that we useEvent Handler objects rather than
functions, the next step is to define the interface of the
Event Handler. There are two approaches:
A single-method interface: The OMT diagram in Section 7 illustrates an implementation of the Event
Handler base class interface that contains a single
method, called handle event, which is used by the
Initiation Dispatcher to dispatch events. In this
case, the type of the event that has occurred is passed as a
parameter to the method.
The following C++ abstract base class illustrates the
single-method interface:

```c++
class Event_Handler
// = TITLE
// Abstract base class that serves as the
// target of the Initiation_Dispatcher.
{
public:
// Hook method that is called back by the
// Initiation_Dispatcher to handle events.
virtual int handle_event (Event_Type et) = 0;
// Hook method that returns the underlying
// I/O Handle.
virtual Handle get_handle (void) const = 0;
};
```

The advantage of the single-method interface is that it is
possible to add new types of events without changing the interface. However,this approach encourages the use of switch
statements in the subclass’s handle event method,which
limits its extensibility.
A multi-method interface: Another way to implement the
Event Handler interface is to define separate virtual hook methods for each
type of event (such as handle input, handle output,
or handle timeout).
The following C++ abstract base class illustrates the
single-method interface:  

```c++
class Event_Handler
{
public:
// Hook methods that are called back by
// the Initiation_Dispatcher to handle
// particular types of events.
virtual int handle_accept (void) = 0;
virtual int handle_input (void) = 0;
virtual int handle_output (void) = 0;
virtual int handle_timeout (void) = 0;
virtual int handle_close (void) = 0;
// Hook method that returns the underlying
// I/O Handle.
virtual Handle get_handle (void) const = 0;
};  

```

The benefit of the multi-method interface is that it is
easy to selectively override methods in the base class and
avoid further demultiplexing, e.g., via switch or if statements, in the hook method. However, it requires the framework developer to anticipate the set of Event Handler
methods in advance. For instance, the various handle *
methods in the Event Handler interface above are tai-
lored for I/O events available through the UNIX select
mechanism. However, this interface is not broad enough
to encompass all the types of events handled via the Win32
WaitForMultipleObjects mechanism [6].
Both approaches described above are examples of the
hook method pattern described in [3] and the Factory Callback pattern described in [7]. The intent of these patterns is
to provide well-defined hooks that can be specialized by applications and called back by lower-level dispatching code.  
9.5 Determine the Number of Initiation Dispatchers in an Application
Many applications can be structured using just one instance
of the Reactor pattern. In this case, the Initiation
Dispatcher can be implemented as a Singleton [5]. This
design is useful for centralizing event demultiplexing and
dispatching into a single location within an application.
However, some operating systems limit the number of
Handles that can be waited for within a single thread
of control. For instance, Win32 allows select and
WaitForMultipleObjectsto wait forno morethan64
Handles in a single thread. In this case, it may be necessary to create multiple threads, each of which runs its own
instance of the Reactor pattern.
Note that Event Handlers are only serialized within
an instance of the Reactor pattern. Therefore, multiple
Event Handlers in multiple threads can run in parallel.
This configurationmay necessitate the use of additional synchronization mechanisms if Event Handlers indifferent
threads access shared state.  
9.6 Implement the Concrete Event Handlers
The concrete event handlers are typically created by appli-
cation developers to perform specific services in response to

particular events. The developers must determine what pro-
cessing to perform when the corresponding hook method is
invoked by the initiation dispatcher.
The following code implements the Concrete Event
Handlers for the logging server described in Section 3.
These handlers provide passive connection establishment
(Logging Acceptor) and data reception (Logging
Handler).
The Logging Acceptor class: This class is an example
of the Acceptor component of the Acceptor-Connector
pattern [8]. The Acceptor-Connector pattern decouples the
task of service initialization from the tasks performed after a
service is initialized. This pattern enables the application-specific portion of a service, such as the Logging
Handler, to vary independently of the mechanism used to
establish the connection.  
A Logging Acceptor passively accepts connections from client applications and creates client-specific
Logging Handler objects, which receive and process
logging records from clients. The key methods and data
members in the Logging Acceptor class are defined below:  

```c++
class Logging_Acceptor : public Event_Handler
// = TITLE
// Handles client connection requests.
{
public:
// Initialize the acceptor_ endpoint and
// register with the Initiation Dispatcher.
Logging_Acceptor (const INET_Addr &addr);
// Factory method that accepts a new
// SOCK_Stream connection and creates a
// Logging_Handler object to handle logging
// records sent using the connection.
virtual void handle_event (Event_Type et);
// Get the I/O Handle (called by the
// Initiation Dispatcher when
// Logging_Acceptor is registered).
virtual HANDLE get_handle (void) const
{
return acceptor_.get_handle ();
}
private:
// Socket factory that accepts client
// connections.
SOCK_Acceptor acceptor_;
};
```


The Logging Acceptor class inherits from the Event
Handler base class. This enables an application to reg-
ister the Logging Acceptor with an Initiation
Dispatcher.
The Logging Acceptor also contains an instance of
SOCK Acceptor. This is a concrete factory that enables
the Logging Acceptorto accept connectionrequestson
a passive mode socket that is listening to a communication
port. When a connection arrives from a client, the SOCK
Acceptor accepts the connection and produces a SOCK
Stream object. Henceforth, the SOCK Stream object is

The Logging Acceptor class inherits from the Event
Handler base class. This enables an application to reg-
ister the Logging Acceptor with an Initiation
Dispatcher.
The Logging Acceptor also contains an instance of
SOCK Acceptor. This is a concrete factory that enables
the Logging Acceptorto accept connectionrequestson
a passive mode socket that is listening to a communication
port. When a connection arrives from a client, the SOCK
Acceptor accepts the connection and produces a SOCK
Stream object. Henceforth, the SOCK Stream object is

used to transfer data reliably between the client and the log-
ging server.
The SOCK Acceptor and SOCK Stream classes used
to implement the logging server are part of the C++ socket
wrapper library provided byACE[9]. These socket wrappers
encapsulate the SOCK Stream semantics of the socket in-
terface within a portable and type-secure object-oriented in-
terface. In the Internet domain, SOCK Stream sockets are
implemented using TCP.
The constructor for the Logging Acceptor registers
itself with the Initiation Dispatcher Singleton [5]
for ACCEPT events, as follows:  

```c++
Logging_Acceptor::Logging_Acceptor
(const INET_Addr &addr)
: acceptor_ (addr)
{
// Register acceptor with the Initiation
// Dispatcher, which "double dispatches"
// the Logging_Acceptor::get_handle() method
// to obtain the HANDLE.
Initiation_Dispatcher::instance ()->
register_handler (this, ACCEPT_EVENT);
}
Henceforth, whenever a client connection arrives, the
Initiation Dispatcher calls back to the Logging
Acceptor’s handle event method, as shown below:
void
Logging_Acceptor::handle_event (Event_Type et)
{
// Can only be called for an ACCEPT event.
assert (et == ACCEPT_EVENT);
SOCK_Stream new_connection;
// Accept the connection.
acceptor_.accept (new_connection);
// Create a new Logging Handler.
Logging_Handler *handler =
new Logging_Handler (new_connection);
}  

```

The handle event method invokes the accept method
of the SOCK Acceptor to passively establish a SOCK
Stream. Once the SOCK Stream is connected with
the new client, a Logging Handler is allocated dynamically to process the logging requests. As shown
below, the Logging Handler registers itself with the
Initiation Dispatcher, which will demultiplex all
the logging records of its associated client to it.
The Logging Handler class: The logging server uses the
Logging Handler class shown below to receive logging
records sent by client applications:  

```c++
class Logging_Handler : public Event_Handler
// = TITLE
// Receive and process logging records
// sent by a client application.
{
public:

// Initialize the client stream.
Logging_Handler (SOCK_Stream &cs);
// Hook method that handles the reception
// of logging records from clients.
virtual void handle_event (Event_Type et);
// Get the I/O Handle (called by the
// Initiation Dispatcher when
// Logging_Handler is registered).
virtual HANDLE get_handle (void) const
{
return peer_stream_.get_handle ();
}
private:
// Receives logging records from a client.
SOCK_Stream peer_stream_;
};
Logging Handler inherits from Event Handler,
which enables it to be registered with the Initiation
Dispatcher, as shown below:
Logging_Handler::Logging_Handler
(SOCK_Stream &cs)
: peer_stream_ (cs)
{
// Register with the dispatcher for
// READ events.
Initiation_Dispatcher::instance ()->
register_handler (this, READ_EVENT);
}
Once it’s created, a Logging Handler registers itself
for READ events with the Initiation Dispatcher
Singleton. Henceforth, when a logging record arrives,
the Initiation Dispatcher automatically dispatches
the handle event method of the associated Logging
Handler, as shown below:
void
Logging_Handler::handle_event (Event_Type et)
{
if (et == READ_EVENT) {
Log_Record log_record;
peer_stream_.recv ((void *) log_record, sizeof log_record);
// Write logging record to standard output.
log_record.write (STDOUT);
}
else if (et == CLOSE_EVENT) {
peer_stream_.close ();
delete (void *) this;
}
}  
```

When a READ event occurs on a socket Handle,
the Initiation Dispatcher calls back to the
handle event method of the Logging Handler.
This method receives, processes, and writes the logging
record to the standard output ( STDOUT ). Likewise, when
the client closes down the connection the Initiation
Dispatcher passes a CLOSE event, which informs the
Logging Handler to shut down its SOCK Stream and
delete itself.  

When a READ event occurs on a socket Handle,
the Initiation Dispatcher calls back to the
handle event method of the Logging Handler.
This method receives, processes, and writes the logging
record to the standard output ( STDOUT ). Likewise, when
the client closes down the connection the Initiation
Dispatcher passes a CLOSE event, which informs the
Logging Handler to shut down its SOCK Stream and
delete itself.  

9.7 Implement the Server  
The logging server contains a single main function.
The logging server main function: This function implementsa single-threaded,concurrentloggingserverthat waits
in the Initiation Dispatcher’s handle events
event loop. As requests arrive from clients, the
Initiation Dispatcher invokes the appropriate
Concrete Event Handler hook methods, which accept connections and receive and process logging records.
The main entry point into the logging server is defined as
follows:  

```c++
// Server port number.
const u_short PORT = 10000;
int
main (void)
{
// Logging server port number.
INET_Addr server_addr (PORT);
// Initialize logging server endpoint and
// register with the Initiation_Dispatcher.
Logging_Acceptor la (server_addr);
// Main event loop that handles client
// logging records and connection requests.
for (;;)
Initiation_Dispatcher::instance ()->
handle_events ();
/* NOTREACHED */
return 0;
}  

```

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

