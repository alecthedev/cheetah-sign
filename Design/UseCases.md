# Use Cases

## Actors
- Trust administrator
- Clients

Explain characteristics

## Use Cases

- **UC1**: Manage Documents
	- BR1
	- Administrators
	- **Why**: Managing documents is the core feature of the application besides signing the documents themselves. This means administrators need a way to view, upload, and save frequently used documents.
	- **Flow**:
 		- The GUI will allow the administrators to complete the following:
			- Upload documents
			- Save/browse favorited documents
			- View document history

- **UC2**: Sign Documents
	- BR1
	- Administrators and Clients
	- **Why**: The purpose of this app is to provide a seamless integration of signing documents in Cheetah flow, so it is of utmost priority to include this use case. The entirety of the project depends on the ability to send and receive documents for signatures. Sometimes more than one document needs to be signed, and usually by multiple parties. 
	- **Flow**:
   		- Administrators select documents that have been uploaded, and add areas that need filled out by drag-and-drop. The documents are then sent out to the one party for signatures. The signing order determines the next person that will receive the document, once each signer finishes. 

- **UC3**: Prepare and Send Documents to Clients
	- BR1
	- Administrators
	- **Why**: This use case is the preparation of the documents that need signed by the administrators and for the clients. In order for the forms to be reproducible and not burdensome, the use of pre-filled fields will be used for some items. The admins will also need to mark where signature fields will be placed on a document. 
	- **Flow**:
		- Administrators will be designating data that can be pre-filled, and where they need filled in. This includes names, addresses, dates, phone numbers, etc. They will also designate where signatures are required and other information that will not be filled in automatically, but is required to be filled in the by the recipients. The document is marked "prepared" once all fields are placed, and can notw be sent to clients. 

- **UC4**: Sign and Return Documents to Admin
	- BR1
	- Clients
	- **Why**: This use case is the situation where a document has been created and fields have been marked for signing. It is key to BR1 that the clients are able to sign the documents and then send the document back to admin.
	- **Flow**:
		- Client will access documents that need signed via an email. They will be prompted to sign in the designated fields. When they are finished they can send the document to the next party, likely the administrators. 

- **UC5**: Log Audit Trail of Document Use
	- BR1
	- Administrators
	- **Why**: Due to the financial and legal nature of the domain of this software, it is key that some data be tracked and stored. Timestamps with the date and time will be tracked for where a document is and who it has/has not been signed by. 
	- **Flow**:
		- Within the GUI, an administrator can view when a document has been uploaded/prepared and whether a document has been: 
			- Sent to client
			- Opened by client
			- Signed and returned by the client
 	 	- Administrators will also see when documents are uploaded and prepared, from the GUI. 

- **UC6**: Login System for Cheetah Sign Administrators
	- BR1
	- Administrators
	- **Why**: By creating a login system for administrators we can protect the legally-binding documents involved in the signing process. The nature of this domain requires document security to be taken seriously. 
	- **Flow**: 
		- Users should be able to login with a username and password using the GUI, so that they can access all of their documents and audit trails. 


