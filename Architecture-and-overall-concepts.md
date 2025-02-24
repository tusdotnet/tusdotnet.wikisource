This page describes the architecture of tusdotnet and the overall idea on how everything about it fits together.

There are three main parts of tusdotnet:
* TusMiddleware
* A configuration object
* A data store

## TusMiddleware
This is the OWIN middleware responsible for handling all requests and returning the correct responses. The functionality of the middleware depends on what interfaces are implemented by the data store used.

To setup the middleware, use the `UseTus` extension method and provide it with a configuration factory. Using this approach tusdotnet supports using different configurations for different requests (e.g. different users, tenants etc).

The middleware will forward all requests that it cannot handle, e.g. GET requests or other requests missing e.g. the Tus-Resumable header. This way developers can still handle uploads even if the client that does not support the tus protocol.

## Configuration object
The configuration object tells the TusMiddleware what options to use. It currently supports setting what URL to listen to, what data store to use and custom callbacks to run during the request processing.

See [Configure tusdotnet](Configure-tusdotnet) for more details.

## Data store
The data store is where tusdotnet stores its data and also what determines the functionality of the server running tusdotnet. There are a number of interfaces that the store can implement each giving the system more functionality. tusdotnet ships with `TusDiskStore` which is a simple store that saves data in a folder on the server disk. [Custom data stores](Custom-data-store) are supported. 
