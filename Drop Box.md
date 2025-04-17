**Dropbox** is a cloud storage service which allows users to store their data (files) on remote servers and keeps it secure and durable on remote server which is accessible using multiple devices from anywhere with internet.

**Functional Requirement:**
	1. users should be able to signup and subscribe for a plan, if they don't subscribe then they will get 1 GB of free access.
	2. users can upload/download files from any device
	3. users can add/edit/delete these files on remote server.
	4. users can share  files and folders with other users.
	5. users should be able to upload files upto 1 GB .
	6. System should support automatic syncronization across the devices.
	
**Non Functional Requirement:**
	1. System should be highly reliable and shouldn't lost any data.
	2. should should be highly available
	

**Traffic Assumption and Estimation:**
	
	Dropbox have total 500 million users
	dropbox have 100 million Daily Active USers.
	on an average each user have 200 files.
	
	total number of files = 500 M * 200 = 100000 M = 100 Billion
	
	on an average each file have 100KB data 
	total storage = 500 M * 200 * 100 KB = 500 M * 20000 KB = 10000000 M * KB = 10 T * KB = 10 PB

<img width="514" alt="image" src="https://github.com/user-attachments/assets/3aa8ef1a-45fd-40f1-b655-6891fc016cd4" />


**Core Key Components**

**Cient**: Client is the application running on users device (mobile/desktop) and watches customers workspace and syncronizes the files with remote server.
	1. Watch user workspace for changes.
	2. upload/download files from remote server 
	3. handle the conflicts due to offline or concurrent updates
	4. update the meta data on remote server if thet change

  Suppose user wants to upload a file with some changes on remote server, in this case it require 1 GB storage to keep that file in block storage. imagin user made few more changes which
  require subsequently 1 GB storage for each request, which is time consuming as well as need higher storage, same applies to download those chanes from different clients. to avoid this problem client 
  uses chunker which will split each file into small chunks and keep file/chunk information in client metadata db. during file update only the chunk which got modified will be sent to remote server.
  	
	Client metadata database: client metadata database stores the information about different files in workspace, file chunks, , chunk version and file location in the file system. this can be implemented using a lightweight database like SQLite
	
	Chunker: Chunker splits the big files into chunks of 4mb each. this also reconstructs the original file from chunks.
	
	Watcher: Watcher monitors for file changes in workspace like update, create, delete of files and folders. Watcher notifies indexer about the changes.
	
	Indexer: indexer listens for the events from watcher and updates the client metadata database with information about the chunk of modified file . it also notifies synchronizer after commiting the changes to client metadata database.
	
	Synchronizer: Synchronizer listens for events from indexer and communicates with meta service and block service for updating data and modified chunk of file on remote server respectively. it alos listens for changes broadcasted by notification service and downloads the modified chunk from the remote server.
	
**Meta service:** meta service is responsible for synchronizing the metadata from client to server. its also responsible to figure out the change set for different clients and broadcast them using notification service.

when a client comes online, it pings meta service for an update. meta service determines the chages set for that client by querying the meta db and returns the change set.

if a client updates a file, meta service again determines the change set for other clients watching the file and broadcast the changes via notification service.

Meta service is backed by metadata db, this database contains the metadata of file like name, type, sharing permissions, chunk information etc. this database should have strong ACID properties.

Since querying database for every synchronization request is costly operation, a in-memory cache is put in front of metadata DB. frequent queriy data wil be cahced in the cahce there by eliminating the need of database query.


**Block Service:** block service interacts with block storage for uploading and downloading of files. client connects with block service to upload or download file chunk.

when a client finishes downloading file, block service notifies meta service to update the metadata. when a client uploads a file, block service on finishing the upload to block storage, notifies the meta service to update the meta data corresponding to this client and broadcast messages for other clients.


**Notification service:** notification service broadcasts the file changes to connected clients making sure any change to file is reflected all watching clients instantly.

notification service can be implemented using HTTP long pooling, websocket or server sent event.

notification service before sending the data to clients, reads the message from a message queue.this queue can be implemented using rabbitMQ, Apache kafka .Message queue provides asyncronous medimu of communication b/w meta and notification service.
