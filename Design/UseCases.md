# Use Cases

## Actors
- Trust administrator
- Clients

Explain characteristics

## Use Cases

- UC1: Manage Documents
	- BR1
	- Administrators
	- Why: Managing documents is the core feature of the application besides signing the documents themselves.
	- Flow:
		- Uploading documents
		- Saving/browsing favorited documents
		- Viewing document history

- UC2: Sign Documents
	- BR1
	- Administrators and Clients
	- Why: The purpose of this app is to provide a seamless integration of signing documents in Cheetah flow, so it is of utmost priority to include this use case. 
	- Flow:
		- Multiple signers
		- Multiple documents

- UC3: Prepare and Send Documents to Clients
	- BR1
	- Administrators
	- Why: This use case is the preparation of the documents that need signed by the administrators and for the clients.
	- flow:
		- designating data that can be pre-filled, and where
			- names, addresses, dates
		- Marking where each party will sign

- UC4: Sign and Return Documents to Admin
	- BR1
	- Clients
	- Why:
	- Flow:
		- Client will access documents that need signed via an email. They will be prompted to sign in the designated fields. When they are finished they can send the document to the next party, likely the administrators. 

- UC5: Log Audit Trail of Document Use
	- BR1
	- Administrators
	- Why:
	- Flow:
		- Within the GUI, an administrator can view when a document has been uploaded/prepared and whether a document has been: 
			- Sent to client
			- Opened by client
			- Signed and returned by the client

- UC6: Login System for Cheetah Sign Administrators
	- BR1
	- Administrators
	- Why:
	- Flow: 
		- Users should be able to login with a username and password using the GUI, so that they can access all of their documents and audit trails. 


