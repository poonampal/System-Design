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
