## HTTP - Hypertext Transfer Protocol

[RFC 2616](https://tools.ietf.org/html/rfc2616)

### Overview
* Application protocol for distributed, collaborative, hypermedia systems
	* Foundation of data communication for the web
* Functions as a request-response protocol in client-server computing
* Application layer protocol, by definition assumes an underlying reliable trasport protocol
	* Obviously, TCP is most often used

### Definitions
* **Connection** - A transport layer virtual circuit (socket) 
* **Message** - Basic unit of HTTP communication
* **Request** - an HTTP request message (from an HTTP client) 
* **Response** - an HTTP response message (from an HTTP server)
* **Resource** - a newtwork data object that can be identified by a URI
* **Entity** - Information tranferred as the payload of a request or response
	* Metainformation in the form of entity-header and content in the form of entity-body
* **Client** - a program that established connections for sending requests
* **User agent** - The client which initiates the request (browsers, editors, other end user tools)
* **Server** - An application program that accepts connections in order to service requests by sending back responses
* **Proxy** - An intermediary program which acts as both a server and client
	* Makes requests on behalf of other clients
	* A **transparent proxy** does not modify the request or response beyond what is needed for authorization
	* A **non-transparent proxy** is a proxy that modifies the request or response in order to provide some added service to the user agent
* **Cache** - A program's local store of response messages
* **Upstream/downstream** - describes flow of a messages; messages flow from upstream to downstream

### Request Methods
* **OPTIONS** - request for information about the communication options available on the request/response chain
* **GET** - The GET method means retrieve whatever information (in the form of an entity) is identified by the Request-URI
* **HEAD** - identical to GET except that the server MUST NOT return a message-body in the response
* **POST** - designed to allow a uniform method to cover the following functions:
	* Annotation of existing resources;
	* Posting a message to a bulletin board, newsgroup, mailing list, or similar group of articles;
	* Providing a block of data, such as the result of submitting a form, to a data-handling process;
	* Extending a database through an append operation.
* **PUT** - requests that the enclosed entity be stored under the supplied Request-URI
* **DELETE** - requests that the origin server delete the resource identified by the Request-URI
* **TRACE** - used to invoke a remote, application-layer loopback of the request message
* **CONNECT** - reserved for use with a proxy that can dynamically switch to being a tunnel

### HTTP Response Status Code Basics
* **1xx**: Informational - Request received, continuing process
* **2xx**: Success - The action was successfully received, understood, and accepted
* **3xx**: Redirection - Further action must be taken in order to complete the request
* **4xx**: Client Error - The request contains bad syntax or cannot be fulfilled
* **5xx**: Server Error - The server failed to fulfill an apparently valid request

[List of Status Codes - Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)