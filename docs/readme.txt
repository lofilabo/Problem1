

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

The Data Collection Service must be able to implement interfaces for each of these.  It must be able to consolidate their data requests into a single data request protocol type, and forward the requests to an application service, and it must be able to perform the same 







