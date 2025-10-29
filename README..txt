::: {#about onclick="this.style.display = 'none'"}
about
:::

::: {#modified}
12/08/2022
:::

::: {#page}
CommCompare
:::

::: {#hlp}
:::

[]{#top}

:::: {}
[CommCompare code](https://github.com/JimFawcett/CommCompare){.repoLink
target="_blank"}

# CommCompare┬á┬áRepository {#title}

### Comparing C++ and Rust Message-passing communicators {#subtitle .indent}

::: {onmouseleave="closeQuickStatus()" style="padding-right:25px; position:absolute; top:1.9em; right:0.25em;"}
Quick Status

Code functions correctly No known defects Demonstration code yes
Documentation yes Test cases yes Static library yes Build requires Rust
and C++ installation Planned changes None at this time
:::
::::

## Comparing C++ and Rust {#compare}

My collaborator, Mike Corley, and I are interested in Rust and how it
compares with the C++ programming language. To do this in a practical
setting, we decided to implement message-passing communicators using
socket-based connections and simple variable-sized messages. The results
of that are what you find in this repository. Here, we will compare
performance, complexity, ease of construction, code size, and safety.
For some of these attributes, like performance, we can be quite
specific. For others, like ease of construction, we will be satisfied
with subjective judgments. We established several goals for this
demonstration:

1.  The C++ and Rust designs should be as similar as practical within
    constraints of the two languages. Both use:
    - Synchronous, full-duplex sockets for client to server
      client-handler communication.
    - Queues for message input in both client and server.
    - Server Threadpools to avoid context thrashing for large numbers of
      concurrent clients.
    - Simple variable-size messages which may have binary bodies.
    - Server processing simply echos client messages back (client
      handlers are designed to support significantly more complex server
      operations).
2.  These projects will use each language\'s standard libraries for
    programming resources, but will not use other third party libraries
    like Boost and those from Crates.io. The intent is to compare our
    work with C++ and Rust, not some other developer\'s work.
3.  The C++ code should use modern C++17 constructs supporting safe data
    handling. Safe Rust enforces memory and data race safety at compile
    time. No unsafe blocks will be used in any of the Rust code
    (excluding unsafe blocks used in, and thoroughly vetted for, the
    standard library).
4.  The code should provide build processes for Windows, Linux, and
    (eventually) macOS.
5.  Implementations will use professional care, but no extraordinary
    efforts will be made to squeeze out additional performance or other
    measured attributes. This is supposed to reflect good \"business as
    usual\" practice.

------------------------------------------------------------------------

### Communicator Concepts {#concept}

:::::::::: {}
::::::::: {style="display:flex; flex-wrap:nowrap;"}
### C++ Communicator

:::: {}
Message-Passing Library (MPL) provides message-passing communication
between a TCPConnector and TCPResponder.

::: {}
[ Fig 1. MPL Concept ]{style="
          display: inline-block;
          font-weight: bold;
          font-family: 'Comic Sans MS', Tahoma;
          background-color: #ddd;
          width: 100%;
          padding: 5px 0px;
          "}
:::

It uses TCPClientSocket and TCPServerSocket classes to establish
synchronous bilateral connections between instances of TCPConnector and
ClientHandler. Fig. 2 illustrates a typical set of sessions between 16
TCPConnector clients and their server-side ClientHandlers. Clients send
1000 4K messages concurrently with the other clients.
::::

::: {}
[ Fig 2. MPL Output Test4 ]{style="
            display: inline-block;
            font-weight: bold;
            font-family: 'Comic Sans MS', Tahoma;
            background-color: #ddd;
            width: 100%;
            padding: 5px 0px;
          "}
:::

Each client has a sending thread and a receiving thread, so the client
does not wait for a reply before sending its next message.

### Rust Communicator

:::: {}
RustComm is a facility for sending messages between a Connector and
Listener.

::: {}
[ Fig 1. RustComm Concept ]{style="
            display: inline-block;
            font-weight: bold;
            font-family: 'Comic Sans MS', Tahoma;
            background-color: #ddd;
            width: 100%;
            padding: 5px 0px;
            "}
:::

It uses the std::net::TcpStream and std::net::TcpListener types to
establish communication between Connector and Listener::Process
components. Listener::Process plays the same role as the MPL
ClientHandler, handling message transactions between the Listener and
its associated Connector. For each incoming connection the Listener
dispatches a threadpool thread to execute Listener::Process code for the
associated Connector.
::::

::: {}
[ Fig 2. RustComm Output Test4 ]{style="
            display: inline-block;
            font-weight: bold;
            font-family: 'Comic Sans MS', Tahoma;
            background-color: #ddd;
            width: 100%;
            padding: 5px 0px;
          "}
:::

Fig 2. illustrates communication sessions for 16 clients and their
associated Listener::Process components. As with MPL, clients send 1000
4K messages while other clients are also sending. Each client has a
sending and a receiving thread, so the client does not wait for a reply
before sending the next message.
:::::::::
::::::::::

------------------------------------------------------------------------

### Communicator Comparisons {#comp}

1.  #### Performance:

    Performance is a very important issue for most C++ and Rust
    developers. In this section we compare performance of the two
    message-passing systems described above. One is implemented using
    C++17 and the other using Rust 1.48. The designs of each are as
    close as practical, given the syntax and semantics of the two
    languages. We measured performance by message-passing throughput in
    MegaBytes per Second for 16 concurrent clients each passing 1000
    messages to a server that that uses a threadpool running client
    handlers that simply echo back each message. Clients send on one
    thread, receive on another, and don\'t wait for replies before
    sending the next message. Throughput is measured as the total
    message content bytes for all clients divided by the time between
    sending the first message and receiving the last. Here are the
    results for six different environments, e.g., three different
    desktops, two operating systems, running on both the native platform
    and in a VMware virtual machine.

    ::::: {.indents .pad3 style="width:calc(100vw - 10em); margin-top:1.5em;"}
    RustComm 5.20% faster than MPL on Windows - Core i7-7700

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Intel Core i7-7700 CPU, 16 GB RAM, 932GB SSD,                       |
    | 64b┬áWindows┬á10┬áver┬á2004, cl ver 19.28                               |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 202     | 218     | 229     | 230     | 195     | 217     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 215.2 MB/s                                             |
    +---------------------------------------------------------------------+

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Intel Core i7-7700 CPU, 16 GB RAM, 932GB SSD,                       |
    | 64b┬áWindows┬á10┬áver┬á2004, Rustc 1.47.0                               |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 227     | 248     | 229     | 214     | 235     | 209     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 227.0 MB/s                                             |
    +---------------------------------------------------------------------+
    :::::

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    C++ MPL 9.32% faster than RustComm on Ubuntu - Core i7-2600

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Intel Core i7-2600 CPU, 8 GB RAM, 1.5 TB ATA,                       |
    | 64b┬áUbuntu┬áversion┬á20.04, gcc version 9.3.0                         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 610     | 640     | 626     | 622     | 610     | 785     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 648.8 MB/s                                             |
    +---------------------------------------------------------------------+

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Intel Core i7-2600 CPU, 8 GB RAM, 1.5 TB ATA,                       |
    | 64b┬áUbuntu┬áversion┬á20.04, Rustc 1.47.0                              |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 671     | 608     | 606     | 618     | 456     | 571     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 588.3 MB/s                                             |
    +---------------------------------------------------------------------+
    :::::

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    RustComm 17.2% faster than MPL on Windows - Xeon Workstation

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áWindows 10 Workstation┬á, cl        |
    | version 19.28.29334                                                 |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 395     | 438     | 385     | 398     | 422     | 391     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 404.8 MB/s                                             |
    +---------------------------------------------------------------------+

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áWindows 10 Workstation┬á, Rustc ver |
    | 1.48.0                                                              |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 499     | 513     | 434     | 495     | 486     | 507     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 489.0 MB/s                                             |
    +---------------------------------------------------------------------+
    :::::

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    MPL 4.21% faster than RustComm on Windows - Xeon Workstation, VMware
    VM

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áWindows 10 on VMware workstation   |
    | 16.1.0┬á, cl version 19.28.29334                                     |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 355     | 424     | 435     | 411     | 418     | 436     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 413.2 MB/s                                             |
    +---------------------------------------------------------------------+

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áWindows 10 on VMware workstation   |
    | 16.1.0┬á, Rustc ver 1.48.0                                           |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 383     | 421     | 411     | 373     | 373     | 414     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 395.8 MB/s                                             |
    +---------------------------------------------------------------------+
    :::::

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    MPL 14.8% faster than RustComm on Linux - Xeon Workstation, VMware
    VM

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áLinux Mint ver 20 on VMware        |
    | workstation 16.1.0┬á, g++ 9.3                                        |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 630     | 640     | 639     | 600     | 607     | 614     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 621.7 MB/s                                             |
    +---------------------------------------------------------------------+

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    +---------+---------+---------+---------+---------+---------+---------+
    | Xeon E3-1535M v6, 32 GB RAM, 64b┬áLinux Mint ver 20 on VMware        |
    | workstation 16.1.0┬á, Rustc ver 1.48.0                               |
    +---------+---------+---------+---------+---------+---------+---------+
    | Thruput | 522     | 517     | 633     | 439     | 492     | 574     |
    | MB/s    |         |         |         |         |         |         |
    | with 8  |         |         |         |         |         |         |
    | thrds,  |         |         |         |         |         |         |
    | 16      |         |         |         |         |         |         |
    | clients |         |         |         |         |         |         |
    +---------+---------+---------+---------+---------+---------+---------+
    | Avg Thruput: 529.5 MB/s                                             |
    +---------------------------------------------------------------------+
    :::::

    If you want to reproduce these results, you will find the code here:

    - [RustComm_VariableSizeMsg, master
      branch](https://github.com/JimFawcett/RustCommExperiments/tree/master/RustComm_VariableSizeMsg)
    - [MPL, no-shared-ptr-rust-message-compatibility
      branch](https://github.com/mwcorley79/MPL/tree/no-shared-ptr-rust-message-compatibility)

    with build instructions in Readme.md files in the repository roots.

    #### Summary and Conclusions:

    The performance results are similar, over the several environments
    used. There aren\'t many surprises here, faster processors yield
    faster performance, throughput on Linux is faster than throughput on
    Windows. The conclusion is that C++ and Rust have very similar
    performance, both very good.

2.  #### Size and Complexity {#cmplx}

    In this item we compare the two communicators in terms of size of
    the source code and complexity measured by the number of scopes in
    source codes used to build the communicator programs and also by the
    number of functions in that code. Here are the results:

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

      ----------------------------- ------
      Size - Lines of Source Code     5153
      Number of Functions              266
      Number of Scopes                 544
      ----------------------------- ------

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

      ----------------------------- ------
      Size - Lines of Source Code     2313
      Number of Functions              146
      Number of Scopes                 402
      ----------------------------- ------
    :::::

    This analysis includes, along with implementing code, codes we used
    to test MPL and RustComm. Test code is an integral part of
    development and maintenance and, as such, should be counted as part
    of its size and complexity. Also, analysis includes not only the
    declarative and executable statements, but white space and comments
    that make the code readable and maintainable. That is, the lines of
    code count everything. The data above are affected by the fact that
    Rust has a fairly high level TCPStream library while C++, as of
    C++20, has no networking library. The MPL code used platform socket
    APIs to develop a solid library at the same level of abstraction as
    the Rust TCPStream library. The abstraction implementation had to
    use the slightly different platform socket APIs on Windows and
    Linux. We chose to include the C++ socket library since it is code
    that had to be developed to complete the C++ MPL project. The result
    of this is that we judge Rust to result in somewhat smaller and
    simpler code, mostly because of the scope of its standard libraries.

3.  #### Ease of Construction: {#constr}

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    This comparison is based on personal experience developing C++
    programs for years and more than one year of experience with Rust.
    Treat these comments as opinion, not evidence-based fact.

    ::: {style="font-weight:bold; padding:0.25em;"}
    C++
    :::

    ------------------------------------------------------------------------

    For experienced developers, getting a C++ project to compile is
    usually relatively easy, with the possible exception of template
    metaprogramming code. Developers tend to spend most of their time
    debugging operations, especially for multi-threaded code. Modern C++
    constructs have improved memory safety significantly, but for large
    systems there are likely to be a few places where modern idioms are
    not followed, resulting in errors that are hard to find and fix.

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust
    :::

    ------------------------------------------------------------------------

    The Rust compiler uses static analysis to provide memory and data
    race safety by construction. That means that developers spend
    significantly more time getting complex programs to compile than
    they would with C++. Compiler error messages are very helpful, so
    this isn\'t as difficult as it would otherwise be. Rust developers
    tend to spend much less time debugging code, compared with C++,
    because Rust compiler analysis has eliminated all memory access and
    data race errors, leaving only logical errors with program
    operations.
    :::::

4.  #### Safety:

    Modern C++ is memory safe by convention. It has facilities and
    common idioms that eliminate most memory use errors, as long the
    conventions are followed everywhere. Rust is memory safe by
    construction and is also data race safe by construction. The
    compiler ensures that there are no opportunities for memory and data
    race errors.

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    ------------------------------------------------------------------------

    C++17 has facilities to ensure data safety by convention:
    - range-based for loops prevent indexing outside collections
    - iterators prevent dangling pointers when containers reallocate
      memory
    - std::unique_ptr and std::shared_ptr provide deallocation when they
      go out of scope

    However, it can be difficult to ensure that in large systems all
    code satisfies those conventions. C++ has locks to avoid data races
    by protecting blocks of code, but provides no guarentees that they
    are used correctly, e.g., do all threads share the same lock, are
    locks released in the presence of errors, \...

    ------------------------------------------------------------------------

    During construction of MPL a significant portion of the development
    time was spent finding data races.

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    ------------------------------------------------------------------------

    Rust guarentees memory saftey by construction:
    - Indexing out-of-bounds an array or container causes an ordered
      shutdown, called a panic, that prevents accessing unowned memory.
    - Rust enforces data ownership at compile-time using a reference
      checker that ensures references to data:
      - are initialized before use.
      - do not out live their referends.
      - do not view data mutated by the owner or other references.

    Code that does not conform will fail to build, ensuring memory
    saftey even for large systems. Rust locks protect data, not sections
    of code, which gaurentees that all threads accessing the same data
    use the same lock. That, combinded with Rust\'s data ownership
    rules, avoid data races.

    ------------------------------------------------------------------------

    When implementing Rust Comm, it took care to create multi-threaded
    code that compiled successfully. But once built there were no data
    races to find. So, its harder to build, but much easier to debug
    multi-threaded code in Rust.
    :::::

5.  #### Ease of Maintenance: {#maint}

    Both MPL and RustComm are well commented, factored into single-focus
    components, and have effective build environments. Those factors are
    all part of a good maintenance strategy. In this block we will focus
    on those maintenance attributes that are largely determined by the
    C++ and Rust languages and their environments.

    ::::: {.indents .pad3 style="width:calc(100vw - 10em);"}
    ::: {style="font-weight:bold; padding:0.25em;"}
    C++ MPL
    :::

    ------------------------------------------------------------------------

    The existence and longevity of many large software systems written
    with C++ demonstrate that, if well designed, C++ code can be
    effectively maintained. Unlike Rust, C++ does not have a lot of test
    facilities delivered as part of its default environment. That means
    there is a tendency to test in batches periodically rather than
    continuously during construction. One other issue with C++ is its
    use of header file #includes. Builds using these includes work well,
    but if a project gets deployed to a different directory structure,
    there is a painful process of correcting include links before the
    project builds again.

    ::: {style="font-weight:bold; padding:0.25em;"}
    Rust Comm
    :::

    ------------------------------------------------------------------------

    The Rust ecosystem has a tool called cargo. It is a package and
    build manager, an executor, and it provides access to a linter,
    clippy, and documentor, rustdoc. This tool chain makes maintenance
    of Rust programs very productive. One of its best features is the
    facilities it provides for test. Cargo builds libraries with a
    preconfigured test hook that makes it easy to build unit tests as
    library construction proceeds. In addition to that, cargo will build
    any console app it finds in a /examples directory and link to the
    package library. So it is easy to provide a number of test and
    demonstration programs that illustrate library facilities.
    :::::

------------------------------------------------------------------------

### Implementations {#impl}

:::::::::::: {}
### C++ Communicator {#cppcomm}

::::::: {}
Unlike Rust, C++ does not have a networking library yet (probably coming
with C++2023). So MPL implements a networking library with three main
classes, TCPConnector, TCPResponder, and TCPSocket. These take care of a
lot of the low-level things that are hard to implement cleanly. The MPL
library:

- Uses queued full-duplex buffered message transfers.
- Each message has a fixed size header and a body consisting of an array
  of bytes.
- For each incoming connection the TCPResponder requests a threadpool
  thread and processes messages with an instance of ClientHandler.
- For this demonstration ClientHandler instances echo back the incoming
  message, marked as reply.

The long-term goal for MPL Comm is to serve as a lightweight, flexible,
and performant messaging middleware for systems that require simple and
robust end to end messaging. The TCPConnector and TCPResponder each
provide an object-oriented interface that gives users ability to define
application specific message processing. Additionally, The TCP socket
library is an efficient and portable C++ wrapper around native TCP
socket APIs available on Windows and Linux. This library can be
extracted and used on Windows and Linux as an alternative the native
(non-portable platform specific) APIs.

### Current Design:

There are user-defined types Message, TCPConnector, TCPResponder,
ClientHandler, and TCPSocket. TCPConnector and ClientHandler have
derived classes for specific message types, which in this demonstration
are: VariableSizeConnector and VariableSizeClientHandler. TCPSocket has
derived classes TCPClientSocket nad TCPServerSocket.

------------------------------------------------------------------------

::: pad10
Messages consist of a header with mtype and content_len attributes and
an array of bytes for the body.
:::

------------------------------------------------------------------------

**Message methods:**

1.  **Message(usize sz, const u8 content_buf\[\], usize content_len, u8
    mtype)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Message from array of bytes
    :::
2.  **Message(usize sz, const std::string& str, u8 mtype)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Message from string
    :::
3.  **Message(MessageType mtype=MessageType::DEFAULT)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Message with no contents
    :::
4.  **Message(const Message &msg)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Copy constructor
    :::
5.  **Message &operator=(const Message &msg)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Copy assignment
    :::
6.  **Message(Message &&msg)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Move constructor
    :::
7.  **Message &operator=(Message &&msg)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Move assignment
    :::
8.  **\~Message()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Destructor
    :::
9.  **u8 operator\[\](int index) const**

    ::: {style="padding:3px 10px 5px 10px;"}
    Const index operator
    :::
10. **void set_type(u8 mt)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Set MessageType to one of TEXT, BYTES, STRING
    :::
11. **unsigned get_type() const**

    ::: {style="padding:3px 10px 5px 10px;"}
    Returns instance of MessageType
    :::
12. **usize get_content_len() const**

    ::: {style="padding:3px 10px 5px 10px;"}
    Returns length of message body in bytes
    :::
13. **void set_content_bytes(const u8 buff\[\], size_t len)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Copy byte array into message body
    :::
14. **void set_content_str(const std::string &str)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Copy string to message body
    :::
15. **std::string get_content_str()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Return message body as string
    :::

------------------------------------------------------------------------

::: pad10
TCPConnector supports direct and queued messaging with a connected
TCPResponder.
:::

------------------------------------------------------------------------

TCPConnector methods:

1.  **TCPConnector(TCPSocketOptions \*sc = nullptr);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new TCPConnector
    :::
2.  **bool Close()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Shutdown TCPConnector and signal server-side ClientHandler to
    shutdown.
    :::
3.  **bool IsConnected() const;**

    ::: {style="padding:3px 10px 5px 10px;"}
    Has valid connection?
    :::
4.  **bool IsSending() const;**

    ::: {style="padding:3px 10px 5px 10px;"}
    Is send thread running?
    :::
5.  **bool IsReceiving() const;**

    ::: {style="padding:3px 10px 5px 10px;"}
    Is receive thread running?
    :::
6.  **void UseSendReceiveQueues(bool use_qs);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Starts dedicated send and receive threads.
    :::
7.  **void UseSendQueue(bool use_q);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start send thread on connection established.
    :::
8.  **void UseReceiveQueue(bool use_q);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start receive thread on connection established.
    :::
9.  **void PostMessage(const Message &m);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Enqueues message for sending to associated TCPResponder.
    :::
10. **void SendMessage(const Message &m);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Sends message directly instead of posting to send queue.
    :::
11. **Message GetMessage()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Dequeue received message if available, else block.
    :::
12. **Message ReceiveMessage()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Read message from socket if available, else block.
    :::

------------------------------------------------------------------------

::: pad10
TCPResponder uses threadpool threads to support concurrent communication
sessions.
:::

------------------------------------------------------------------------

**TCPResponder methods:**

1.  **TCPResponder(const EndPoint& ep, TCPSocketOptions\* sc =
    nullptr)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new TCPResponder
    :::
2.  **void RegisterClientHandler(ClientHandler\* ch);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Register ClientHandler prototype - defines server-side message
    processing.
    :::
3.  **void Start(int backlog=20)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start dedicated server listening thread.
    :::
4.  **void Stop()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Stop listening service.
    :::
5.  **void UseClientSendReceiveQueues(bool use_qs);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start dedicated send and receive threads when extablishing a
    connection.
    :::
6.  **void UseClientSendQueue(bool use_q);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start dedicated send thread when establishing a connection.
    :::
7.  **void UseClientReceiveQueue(bool use_q);**

    ::: {style="padding:3px 10px 5px 10px;"}
    Start dedicated receive thread when establishing a connection.
    :::

------------------------------------------------------------------------

::: pad10
ClientHandlers communicate directly with associated TCPConnectors.
Derived ClientHandler classes define application specific message
handling operations.
:::

------------------------------------------------------------------------

**ClientHandler methods:**

1.  **Message GetMessage()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Dequeue message from receive queue if available, else blocks.
    :::
2.  **Message ReceiveMessage()**

    ::: {style="padding:3px 10px 5px 10px;"}
    Read message directly from socket if available, else blocks.
    :::
3.  **void PostMessage(const Message& m)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Enqueues message for connected client.
    :::
4.  **void SendMessage(const Message& m)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Write message directly to connected socket.
    :::
5.  **virtual void AppProc() = 0**

    ::: {style="padding:3px 10px 5px 10px;"}
    Application message processing supplied by derived ClientHandler
    class.
    :::
6.  **virtual ClientHandler\* Clone() = 0;**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create application defined ClientHandler instances.
    :::
:::::::

### Rust Communication {#rustcomm}

::: {}
RustComm is a facility for sending messages between a Sender and
Receiver. It uses the std::net::TcpStream and std::net::TcpListener
types. This is a prototype for message-passing communication system. It
provides three user defined types: Connector, Listener, and Message,
with generic parameters M, P, and L, as shown in Fig. 1. M implements
the Msg trait and represents a message to be sent between endpoints. P
implements the Process\<M\> trait that defines message processing, and L
implements the Logger trait that supports logging events to the console
that can be turned on or off by the types supplied for L, e.g.,
VerboseLog and MuteLog. The RustComm library:

- Uses queued full-duplex buffered message sending and receiving
- Each message has a fixed size header and Vec\<u8\> body.
- For each Connector\<P, M, L\> connection, Listener\<P, L\> processes
  messages until receiving a message with MessageType::END.
- Listener\<P, L\> requests a thread from ThreadPool\<P\> for each
  client connection and processes messages in P::process_message.
- In this version, P::process_message echos back message with \"reply\"
  appended as reply to sender. You observe that behavior by running
  test1, e.g., cargo run \--example test1.

The long-term goal for RustComm is to serve as a prototyping platform
for various messaging and processing strategies. This version defines
traits: Sndr\<M\>, Rcvr\<M\>, Process\<M\>, Msg, and Logger. The
user-defined types, M and P, are things that change as we change the
message structure, defined by M and connector and listener processing
defined by P. These types are defined in the rust_comm_processing crate.
The somewhat complex handling of TcpStreams and TcpListener are expected
to remain fixed. They are defined in the crate rust_comm. Finally,
logger L provides a write method that will, using VerboseLog for L,
write its argument to the console. MuteLog simply discards its argument.
:::

RustComm uses a threadpool, as shown in Fig. 1., to avoid context
trashing for a large number of concurrent clients. In Fig 2. we show
message processing for 16 clients on a machine with 4 hyper-threaded
processors. Each client sends a stream of 1000 4kB messages, all running
concurrently. Timing was executed with a high resolution clock with
microsecond precision. Client sends and receives where executed on
separate threads, so message sending did not wait for message
receptions. Note that the client\'s connector internally blocks waiting
for reply messages, then enqueues them for the client.

### Current Design:

There are three user-defined types: Message, Connector, and Listener.
Connector and Listener each use an existing component
BlockingQueue\<Message\>

::::: pad5
**Message:** Methods:

1.  **new() -\> Message**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Message with empty body and MessageType::TEXT.
    :::
2.  **set_type(&mut self, mt: u8)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Set MessageType member to one of: TEXT, BYTES, END.
    :::
3.  **get_type(&self) -\> MessageType**

    ::: {style="padding:3px 10px 5px 10px;"}
    Return MessageType member value.
    :::
4.  **set_body_bytes(&mut self, b: Vec\<u8\>)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Set body_buffer member to bytes fromb: Vec\<u8\>.
    :::
5.  **set_body_str(&mut self, s: &str;)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Set body_buffer member to bytes froms: &str.
    :::
6.  **get_body_size(&self) -\> usize**

    ::: {style="padding:3px 10px 5px 10px;"}
    Return size in bytes of body member.
    :::
7.  **get_body(&self) -\> &Vec\<u8\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Return body_buffer member.
    :::
8.  **get_body_str(&self) -\> String**

    ::: {style="padding:3px 10px 5px 10px;"}
    Return body contents as lossy String.
    :::
9.  **clear(&self)**

    ::: {style="padding:3px 10px 5px 10px;"}
    clear body contents.
    :::

------------------------------------------------------------------------

::: pad10
Both Connector\<P, M, L\> and Listener\<P, L\> are parameterized with L,
a type satisfying a Logger trait. The package defines two types that
implement the trait, VerboseLog and MuteLog that allow users to easily
turn on and off event display outputs. Fig 2. uses MuteLog in both
Connector\<P, M, L\> and Listener\<P, L\>.
:::

------------------------------------------------------------------------

**Connector\<P, M, L\>** methods:

1.  **new(addr: &\'static str) -\>
    std::io::Result\<Connector\<P,M,L\>\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Connector\<P,M,L\> with running send and receive threads.
    :::
2.  **is_connected(&self) -\> bool**

    ::: {style="padding:3px 10px 5px 10px;"}
    is connected to addr?.
    :::
3.  **post_message(&self, msg: M)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Enqueues msg to send to connected Receiver.
    :::
4.  **get_message(&mut self) -\> M**

    ::: {style="padding:3px 10px 5px 10px;"}
    Reads reply message if available, else blocks.
    :::
5.  **has_message(&self) -\> bool**

    ::: {style="padding:3px 10px 5px 10px;"}
    Returns true if reply message is available.
    :::

**Listener\<P, L\>** methods:

1.  **new(nt: u8) -\> Listener\<P, L\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create new Listener\<P, L\> with nt threads running.
    :::
2.  **start(&mut self, addr: &\'static str) -\>
    std::io::Result\<JoinHandle\<()\>\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Bind Listener\<P,L\> to addr and start listening on dedicated
    thread.
    :::

------------------------------------------------------------------------

::: pad10
CommProcessing\<L\> is parameterized with L, a type satisfying a Logger
trait. The package defines two types that implement the trait,
VerboseLog and MuteLog that allow users to easily turn on and off event
display outputs.
:::

------------------------------------------------------------------------

**CommProcessing\<L\>** methods:

1.  **new() -\> CommProcessing\<L\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Create instance of CommProcessing\<L\>
    :::
2.  **send_message(msg:&M, stream:&mut\<TcpStream) -\>
    std::io::Result\<()\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Write message to TcpStream connected to Connector\<P,M,L\>
    :::
3.  **buf_send_message(msg:&M, stream:&mut┬áBufWriter\<TcpStream\>) -\>
    std::io::Result\<()\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Write message to TcpStream connected to Connector\<P,M,L\>
    :::
4.  **recv_message(stream:&mut┬áTcpStream) -\> std::io::Result\<M\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Read message from TcpStream connected to Connector\<P,M,L\>
    :::
5.  **buf_recv_message(stream:&mut┬áBufReader\<TcpStream\>) -\>
    std::io::Result\<M\>**

    ::: {style="padding:3px 10px 5px 10px;"}
    Read message from TcpStream connected to Connector\<P,M,L\>
    :::
6.  **process_message(msg:&mut┬áM)**

    ::: {style="padding:3px 10px 5px 10px;"}
    Process message using mutable reference.
    :::
:::::
::::::::::::

::: clear
:::

### Build:

You will find code discussed in this documentation in the two
repositories:

- [MPL
  Repository](https://github.com/mwcorley79/MPL/tree/no-shared-ptr-rust-message-compatibility)
- [RustComm
  Repository](https://github.com/JimFawcett/RustCommExperiments)

Build instructions are provided in the Readme.md files in each. Note
that the code for MPL was built from the
no-shared-ptr-rust-message-compatibility branch. The code for RustComm
was built from the master branch.

### Status:

This code implements all of the features we intended for comparison.
There are no known defects. Both codes provide prototypes for future
projects, but will be kept here \"as-is\". []{#bottom} ┬á
[bottom](#bottom) [top](#top)

::: {.darkItem .popupHeader style="padding:0.25em 2.0em;" onclick="this.parentElement.style.display='none'"}
Sections
:::