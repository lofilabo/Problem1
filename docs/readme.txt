

The problem is presente as follows (approximate English translation):

"Build a service to provide Corporate Information of Japan ( company name , address , stock prices , etc.)

This service will be connected to by more than 10,000 instances of a client application.  That (client application) is assumed to exist already.  Design a method for collecting and distributing data.

Give two or more designs.  Justify your choice of which one should be used"


PREAMBLE.

In beginning to design a system of this type, more attention is required to what is unsaid in the specification than what is actually stated.  The explicit part of the specification calls for the 'server' part of a conventional-sounding Client-Server application of the kind popularised circa 1990 when small personal computer desktop workstations were networked to file- and database- servers.  However, the intentional ambiguity in the phrasing of the draft specification above means that such a naive approach would not be appropiate for the task in hand.

Reference is explicitly made, for instance, to the fact that the client application already exists; and presumably is well into its own development lifecycle, has probably been tested and debugged in the field, and in any case may not be changed or customised according to the will of the data server architects.  

The strictest definition of a 'client application' would be one constructed to run natively on business-type operating system, gather and post data from a server by means of a vendor protocol such as COM or CORBA, perhaps a mapped drive; or it might be a multi-user application running on a minicomputer or derivative, and providing terminal sevices to users by means of an RS-232 multiplexer serving many VT-100-type terminals.  These terminals may themselves be emulated on a PC Workstation.  

The application may equally be a web browser application using some or many components of HTML and JavaScript, or it may be web browser appication of the 'RIA' type which presents a highly-capable user interface by means of a small plug-in program.  The possibilities are many, and in each case, the means by which the Client program sends and recieves data is preordained.


The draft specification calls for 2 or more solutions to the problem of how to store, collect and distribute data.  In fact, the initial task probably calls for significantly more than 2, since it is necessary to consider the following requirements:

	* what budget is available for the project?

	* what level of third party system audit is the completed system required to pass?

	* what amount of ongoing development is expected to be done to the system once completed?

	* what proportion of Data Collection and Data Distribution requests are expected?

	* whether or not full transactional support is required at the Data Collection stage.





To address the problem in hand, to begin with, we must separate out the parts of the proposed system that will remain constant whatever the details of the system may be.  These are deemed to be:

	1. a method of interacting with the existing client application	

	2. a means of persistant, reliable data storage.  In almost all cases this will be a Database Server of some kind (with one important exception, discussed later.)

	3. a means of providing data backup

We will discuss each of these in turn.  

1. Data Collection service.

As we have seen, the specification does not state what manner of 'client application' will be consuming data services.  It does not infer what protocol or transportation layer will be used.  We may therefore only briefly sketch the 'client facing' side of this service.  The client facing side must be based around an operating system which can be connected to by any kind of client device, including but not limited to:
	
	Software clients using DCOM type deep-protocol calls
	Software clients using simple TCP / IPX calls
	Terminals connected via an RS-232 type multiplexer
	Thin Clients using some very high-level protocol such as HTTP or SOAP
	Electromechanical devices such as heat sensors and simple-text displays of the 'hotel stock ticker' variety.

The Data Collection Service must be able to implement interfaces for each of these.  It must be able to consolidate their data requests into a single data request protocol type, and forward the requests to an application service, and it must be able to perform the same.

For the purposes of this example, the requests will be made by REST and the data returned as JSON.  This has distinct advantages which should be evident to the reader, but may not be suitable for all situations.  Depending on the strict definition of the application layer, Java RPC calls, XML/SOAP call or Adobe AMF calls may be required.  But the system has been sketched to make the replacement of this step relatively simple.


Option 1.

Please refer to fig 3 in the grfx directory.  This presents a very conservative server architechture of the kind often implemented in exterprise data centres.  We will discuss the nomenclature and function of each part in a little detail.

i. Load Balancer(s).

These of of standard commercial type with no customisation required or desired.  Their function is to distribute incoming and outgoing messages to and from (in the case of LB1) the pair of application servers.  In the case of LB2, the purpose is to route data requests to one of 2 database servers.

ii. Application Servers.

An industry standard in wide use at present is for Application Servers to use either AIX or RHEL, running on some kind of x86 hardware.  The application server itself is often of the Glassfish / Blowfish type, to run applications written in the Java programming language.  Other software vendors of course have comparible offerings involving some kind of Windows server and an appliaction framework for the .NET stack.

The decision of which to employ should not be made lightly.  There is no compatibility between the Unix/JAS/Java and WinNT/.NET application stack, and the latter offers limited upgrade paths to very large server architechtures.

The application servers are duplicated but not load balanced or multiplexed between themselves.  This is not necessary.


iii. Database Servers.

In the server archtechture we are considering, it would be usual to use a large commercial DBMS of some sort.  MS-SQL Server, IBM DB-2 or some Oracle product are the most usual options although others are available.

Typical of this DBMS architechture is the intermediary of a replication server.  This manages the replication of data between two (or more) servers such that one can be delgated to perform read operations and the other for write operations; in the event that one server is lost, the other can perform both operations, with a significantly reduced capacity.

iv. Backup services.

Backing up the above architechture is a difficult task which requires a dedicated solution unto itself.  This is beyond the scope of this discussion.


Option 2.

Please refer to fig 4 in the grfx directory.  This describes a wholly different server architechture oriented around open-source systems.  It is hoped touse this to form a compare-and-contrast basis for the critique that will form the conclusion of this discussion.

i. No load balancer.

This system is intended to be deployed in a less expensive data hosting facility than that specified in (1).  If load balancing is required, it is expected that this can be deferred to the data infrastructure.

ii. Application Server.

Only one application server is shown in this example.  This is because it is believed that, provided sufficient Disaster Recover is available, the AS irself can be very rapidly failed-over to a redundant system.  In this case, the redundant system functions daily as a Staging and Test server.  In the event of a main AS failure, this service can be rapidly substituted.

The software application layer considered for this version of the architechture could, or course, be some variant of some Java application server as in option 1.  This possibility is not be be rejected.  On the other hand, the freedom to select a different application server allows us to address a very important part of the original specification:
	"This service will be connected to by more than 10,000 
	instances of a client application"

This does not make any statement concerning the size of the data and the frequency of the calls.  Data items such as company name and address will very rarely be changed (written).  Stock price may be written several times per day, but the write speed of these changes is not critical.  What is critical the ability for the system to stack data operations which cannot be executed immediately.

The application server layer suggested to fulfill this requirement is NodeJS.  Following the axiom of "use radical technologies for conservative purposes", it is by no means suggested that NodeJS be used for the actual appliaction components themselves (whichmay be written in languages more suided to the actual operations) but rather as a managment layer.  This will ameliorate NodeJS somewhat suboptimal permormance at scale, and its use of the unpleasant JavaScript derived application programming language, whilse maximising its cief strengths: the ability to operate asynchronously, and to be nonblocking.

In short, using NodeJS as the application server management layer will permit calls to queue in a non-suspending way and be fulfilled in an asynchronous manner (i.e: requests can be made in the order r1, r2, r3, but if r2 turns out to be heavy to fulfill, responses can be made in the order r1, r3...(others)...r2, and the whole application stack is not constrained by FIFO considerations.  This in turn should remove the need for multiple application servers and load-balancing.

iii. Database Server.

Commerial databases have peerless permormance and real-time replication.  This cannot be approached by open-source or community DBMS software.  However, provided that arbitration is not required by the database layer, real-time replication from a master to a slave database is by all means possible with many kinds of free / open-source DBMS.  Arbitration can be carried out at the application layer, with read operations directed to the slave and write operations to the master, with a software failover in the event that either is not available.

A futher advantage which can be leveraged using the application server architechture detailed above is that a rapid-read Key Value Pair storage system can be included to remove the need for many read-only database calls.  This is expecially valuable in "fanout on write" systems which prepare a set of data for each users!' consumption when the data is actually written.  KVP systems are not searchable or indexable, and so are unsuiatble for read operations where an ad hoc database query is required, but when the precise key to the required data is readily known (for instance, if the key is derived from the users' login ID), KVP systems make for very light and fast read operations.

iv. Staging and Deplyment.

Commercial systems as detailed in part 1 are well-known for the diffulculty of deploying new systems, software or system changes.  This is necessarily so, because the durability and immense stability of such systems depend on very high levels of quality contraol being exercised over the installed software.

Open-source systems are not structured in this way.  Al though fig.4 shows a system distributed over 8 physical machines, there is no reason that all 8 services present on the production platform could not be made into a development stack on the same machine, i.e, there is no reason that each developer could not have a small PC running a small-but-perfect copy of the entire stack.

The architecture sketched in fig. 4 integrates a staging / test server, which, it is intended, could take over from the production application server in the event of a failure.  A dedicated deployment server is also included, whose job it is to run a build and deploy system which would typically+
	i. collect new or changed software from a source-control system
	ii. Compile it
	iii. Apply test or production configurations.
	iv. Push the changes to the staging server or production server.


Conclusion.

We have briefly discussed two very different approaches to the same problem.  Whilst it is not posible to make a firm recommendation at this stage, we can introduce new areas of consideration which may help when certain other variables are factored in.

Commercial database and AS systems running on industry-standard datacentre hardware are without doubt more reliable and robust, and more resistant to sudden design, than any other except for OEM mainframe systems.  They also have available infrastructure support which further re-inforces their reliability.  Such infrastructure support maybe a prerequisite for obtaining IT audit or insurance, and this must be investigated before any decision about system architecture is made.  However, the cost of ownership of such systems is astronomical.  Something in the order of JY1M / month should be made available to take into account: salaries, hosting costs, licensing costs and routine maintenance tasks.  Certain database vendors levy extra charges per connection or per transaction.  But if the system is required to adhere to an SLA in excess of 99% uptime and fewer than one software error per month (with an SLA penalty of JY50,000 per infraction), this cost may not be avoidable.

The type of system outlined in section 2 does not suffer from these drawbacks.  Because the individual components are quite small and do not demand a dedicated hardware server, full-stack system (not software) development can be carried out on a single machine, rolled out to a single VPS and thence separated out as capacity demands to anything up to 7 dedicated hosts.

Because the test, deployment and staging systems are integrated into the production system, less application audit is required.  Because the system archtechture is simpler, a significantly simpler backup soltion is required; and because the system can be run on a single machine, Disaster Recovery can be accounted for by constantly keeping a backup system built onto a single server.

The second system provides for significantly more freedom in choice of application programming language and database server choice.  The presence of this section in the specification:

	"( company name , address , stock prices , etc.)"

is suggestive.  The 'etc' infers that no firm database schema is available, which itself commends the use of a schemaless DBMS (such systems were not uncommon in legacy minicomputer systems before the supremacy of RDBMS around 1994, and the very arbitraryness of the description 'corporate data' infers that a very solid schema may be unavailable).  There are two main candidates for this role: CouchDB and MongoDB.  CouchDB especially is built using Erlang, which itself has its origins in the telcommunications world.  The system is built for concurrency and trivial scalability, which further reduces the need for dedicated load balancing between multiple DB servers.  MongoDB suffers from difficulties when arbitrating between master and slave instances if the master (arbiter) is itself lost, but possess more developed ORM layers for many application programming languages.

If a solid schema for the data stored in the database can be obtained, better performance and possibly superior replication can be achieved by adopting a more conventional RDBMS of open-source type. Generally, MySQL is favoured for performance and PostgreSQL for feature richness (if views and stored procedures are needed).  It should be noted that as of 2006, MySQL is not free software and probably requires a licence for any commercial operation. 


COPYRIGHT.

This document and its associated diagrams are the original work of Richard Amphlett, and are made available for fair use for study and research. Permission to use for-gain will likely be unconditionally granted: please email to richardlofi@gmail.com.