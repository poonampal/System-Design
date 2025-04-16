**Netflix** is a subscription based streaming platform which allows its members to watch tv shows and movies on devices connected via internet. It supports multiple devices like android, ios, web and TV.

Before we start diving deeper lets first outline Functional and Non functional requirement which will help us to design system as per the requirement.

**Functional Requirement:**
	1. System should allow user to login and watch videos
	2. System should support user to search videos via tag and title
	3. System should have recommendation and artwork thumnail generation feature
	
**Non Functional Requirement:**
	1. system must be highliy available with low latency
	2. System must be reliable and fault tolerant 
	3. System should be scalable and efficient
	
**Traffic Assumption and Estimation:**

 	**Few Formulas for Calculation**
  	BKMGTP
    	B: Byte : Ten: 10
	K : Kilo : Thousand : 1000
	M: Mega: Million: 1000 0000
	G: Giga: Billion: 1000 000 000
	T: Tera: Trillion: 1000 000 000 000
	P: Peta: Quadrillion: 1000 000 000 000 000

 	k*K = M
	M*K = G
	G*K = T
	T*K = P

 	Seconds in a day = 24h * 60m * 60s = 86400 = ~100,000

	Assuming we have total 1 billion customers and 200 miliion Daily Active Users.
	and on an avereage each active user performs 5 actions every day that way
    	total request per day = 200 M * 5 = 1 Billion per day
		
	If we consider 100:1, read:write ration then
		total write request per day = 1/200 * 1B = 5 M writes per day
		
	total read request per second = 1 Billion / (24 * 60 * 60) =1 billion / ~ 100, 000 = 10k request per second

 	Storage 
		suppose each request will require 100 MB storage on average then 
			5 M write request will require = 5 M * 100 MB = 5M * 100 * 1000 KB = ~500 TB 
			if we store videos for 10 years = 500 TB * 10 * 365 = 500 TB * 10 * ~400 =  ~ 200000 * 10 TB = ~ 2000000 TB = ~ 2000 PB
			
	Bandwidth
		500 TB per day = 500 TB / 100, 000 s = 500 * 1000 GB/100,000 s = 5 GB / s


<img width="760" alt="image" src="https://github.com/user-attachments/assets/31994905-617b-4f50-989e-484e997542a3" />


**Core Components:**

**1. Zulu:** Zulu is a gateway service which receives request from netty client and forwards that request to different filters.

Inbound Filter:  Inbound filter will execute before routing to the origin and can be used for things like authentication, routing and decoration.

Endpoint Filter: can be used to return static response otherwise the buildin proxyendpoint filter will route the request to origin.

Outbound Filter: executes after getting the response from the origin and can be used for metrics, decorating the response to the user or adding custom header.

Advantanges of Zulu Service:
	1. Load Testing
	2. Traffic Sharding
	3. new service testing
	4. Filter bad request


**2. Hystrix:** Hystrix is a latency and fault tolerance service designed to isolate points of access to remote systems. This will allow system to fail gracefully without cascading failure to other services.

	1. number of errors > define error
	2. reject request when thread pool is full
	3. timeout calls > time
	4. fallback to default response

**3. EV Cache:** Netflix stores frequently used data in EV cache which is build on top of memcache servers. EV cache clusters are distributed across multiple zones and each cluster will have multiple nodes and will receive read/write request via EV cache client.  https://github.com/netflix/evcache/wiki


**4. Transcoder:** When a client uploads a new video to Netflix, the platform first validates the content for copyright and other policy violations. Once approved, the original video is divided into multiple chunks, which are then transcoded into approximately 1002 different resolutions and formats. These processed chunks are reassembled into a final version, stored on Amazon S3, and distributed to Open Connect servers. From there, the content becomes accessible via Netflix’s edge servers.


**5. Database:** NEtflix uses mysql for transaction related data like payment to ensure ACID compliance and for other information will be stored in cassandra which is a distributed database and highly scalable and fault tolerant.

**6. Chukwa:** Chukwa collects data from all services and provides tools for data analysis. this also distributes all data to Elastic serach and apache spark via kafka.

**7. Chaos Monkey:** Chaos Monkey is a software tool developed by netflix to test the resilience of IT system by intentionally introducing disruptions.

**8. Titus:** Titus is a container management platform.
	
	
