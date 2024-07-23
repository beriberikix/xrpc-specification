# xRPC 1.0 Specification (draft)

## 1 Overview

xRPC (extensible Remote Procedure Call) is a lightweight RPC protocol designed to support multiple data interchange formats. This specification defines several data structures and the rules around their processing as well as mappings to well-known data interchange formats. It is transport agnostic in that the concepts can be used within the same process, over sockets, over http, over serial links or in many various message passing environments. It is inspired by [JSON-RPC 2.0](https://www.jsonrpc.org/specification) and is compatible with the JSON-RPC specification for JSON-based implementations.

It is designed to be simple!

## 2 Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt).

Different data interchange formats have different type systems and represent different "primitive" and "structured" types differently. xRPC utilizes the types for a given format by creating a native "mapping" to the objects in this specification. See the "Mappings" section for more details.

All member names exchanged between the Client and the Server that are considered for matching of any kind should be considered to be case-sensitive. The terms function, method, and procedure can be assumed to be interchangeable.

The Client is defined as the origin of Request objects and the handler of Response objects.

The Server is defined as the origin of Response objects and the handler of Request objects.

One implementation of this specification could easily fill both of those roles, even at the same time, to other different clients or the same client. This specification does not address that layer of complexity.

## 3 Compatibility

xRPC 1.0 Request objects and Response objects may not work with existing xRPC clients or servers other than version 1.0 or any JSON-RPC implementations. However, it is easy to distinguish implementations and versions as xRPC always has a member named "xrpc" with a String value of "1.0".

## 4 Request Object

A rpc call is represented by sending a Request object to a Server. The Request object has the following members:

**xrpc**

> A String specifying the version of the xRPC protocol. MUST be exactly "1.0".

**method**

> A String containing the name of the method to be invoked. Method names that begin with the word rpc followed by a period character (ex. U+002E or ASCII 46) are reserved for rpc-internal methods and extensions and MUST NOT be used for anything else.

**params**

> A Structured value that holds the parameter values to be used during the invocation of the method. This member MAY be omitted.

**id**

> An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null [1] and Numbers SHOULD NOT contain fractional parts [2]

The Server MUST reply with the same value in the Response object if included. This member is used to correlate the context between the two objects.

[1] The use of Null as a value for the id member in a Request object is discouraged, because this specification uses a value of Null for Responses with an unknown id.

[2] Fractional parts may be problematic, since many decimal fractions cannot be represented exactly as binary fractions in certain data interchange formats.

### 4.1 Notification

A Notification is a Request object without an "id" member. A Request object that is a Notification signifies the Client's lack of interest in the corresponding Response object, and as such no Response object needs to be returned to the client. The Server MUST NOT reply to a Notification, including those that are within a batch request.

Notifications are not confirmable by definition, since they do not have a Response object to be returned. As such, the Client would not be aware of any errors (like e.g. "Invalid params","Internal error").

### 4.2 Parameter Structures

If present, parameters for the rpc call MUST be provided as a Structured value. Either by-position through an Array or by-name through an Object.

* by-position: params MUST be an Array, containing the values in the Server expected order.
* by-name: params MUST be an Object, with member names that match the Server expected parameter names. The absence of expected names MAY result in an error being generated. The names MUST match exactly, including case, to the method's expected parameters.

## 5 Response Object

When a rpc call is made, the Server MUST reply with a Response, except for in the case of Notifications. The Response is expressed as a single Object, with the following members:

**xrpc**

> A String specifying the version of the xRPC protocol. MUST be exactly "1.0".

**result**

> This member is REQUIRED on success.

> This member MUST NOT exist if there was an error invoking the method.

> The value of this member is determined by the method invoked on the Server.

**error**

> This member is REQUIRED on error.

> This member MUST NOT exist if there was no error triggered during invocation.

> The value for this member MUST be an Object as defined in section 5.1.

**id**

> This member is REQUIRED.

> It MUST be the same as the value of the id member in the Request Object.

> If there was an error in detecting the id in the Request object (e.g. Parse error/Invalid Request), it MUST be Null.

Either the result member or error member MUST be included, but both members MUST NOT be included.

### 5.1 Error Object

When a rpc call encounters an error, the Response Object MUST contain the error member with a value that is a Object with the following members:

**code**

> A Number that indicates the error type that occurred.

> This MUST be an integer.

**message**

> A String providing a short description of the error.

> The message SHOULD be limited to a concise single sentence.

**data**

> A Primitive or Structured value that contains additional information about the error.

> This may be omitted.

> The value of this member is defined by the Server (e.g. detailed error information, nested errors etc.).

The error codes from and including -32768 to -32000 are reserved for pre-defined errors. Any code within this range, but not defined explicitly below is reserved for future use. The error codes are nearly the same as those suggested for XML-RPC at the following url: http://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php

| code | message | meaning |
| ---- | ------- | ------- |
| -32700 | Parse error | 	Invalid data was received by the server. An error occurred on the server while parsing. |
| -32600 | Invalid Request |  The data sent is not a valid Request object. |
| -32601 | Method not found	| The method does not exist / is not available. |
| -32602 | Invalid params	| Invalid method parameter(s). |
| -32603 | Internal error	| Internal xRPC error. |
| -32000 to -32099 | Server error	| Reserved for implementation-defined server-errors. |

The remainder of the space is available for application defined errors.

## 6 Batch

To send several Request objects at the same time, the Client MAY send an Array filled with Request objects.

The Server should respond with an Array containing the corresponding Response objects, after all of the batch Request objects have been processed. A Response object SHOULD exist for each Request object, except that there SHOULD NOT be any Response objects for notifications. The Server MAY process a batch rpc call as a set of concurrent tasks, processing them in any order and with any width of parallelism.

The Response objects being returned from a batch call MAY be returned in any order within the Array. The Client SHOULD match contexts between the set of Request objects and the resulting set of Response objects based on the id member within each Object.

If the batch rpc call itself fails to be recognized as an valid format or as an Array with at least one value, the response from the Server MUST be a single Response object. If there are no Response objects contained within the Response array as it is to be sent to the client, the server MUST NOT return an empty Array and should return nothing at all.

## 7 Mappings

TODO

## 8 Examples

TODO

## 9 Extensions

Method names that begin with rpc. are reserved for system extensions, and MUST NOT be used for anything else. Each system extension is defined in a related specification. All system extensions are OPTIONAL.