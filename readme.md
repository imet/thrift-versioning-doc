## Introduction

This repo is meant to illustrate best practices for managing version
compatibility of services based on thrift.  It focuses on an SOA
architecture where you generally run one version of a service
implementation but may have clients with older versions of the service
stubs.

## Terminology

When developing a service implementation, a principle design
consideration is compatibility with clients of the service.  In this
context there is a notion of backward compatibility and forward
compatibility.  Forward compatibility describes a version of the
service that will work with clients of future versions of that
service.  

![Matrix of client/server versions and compatibility](images/matrix.png)

In an SOA environment where you intend to only ever run one version of
a given service we're not going to concern ourselves with forward
compatibility and instead focus on backward compatibility.

Backward compatibility is a quality of a service implementation where
it will work with clients running with an earlier version of the
client libraries.  For Thrift this means clients using an older
version of the stubs will still work with a newer version.

Consider a typical deployment of a given Thrift service.  The
interface is defined in the IDL which is used to generate client stubs
for a client app and a platform specific interface that is implemented
by the service.

![Diagram of a thrift client and server](images/clientserver.png)

If the service implementor now modifies the service interface, a new
version of the client stubs is generated.  But not all clients will
upgrade their stubs immediately.  The new version of the service is
considered *backward compatible* with the earlier generated stubs if
the clients using the old stubs can still connect to the service and
invoke methods successfully.  More specifically, the new verison of
the service is _binary backward compatibile_ with the earlier
versions.

![Diagram of client with old version of stubs](images/bbc.png)

A stronger notion of backward compatibility is *source backward
compatibility*.  This means a service client will be able to upgrade
to the new version by simply replacing it's client stubs with the
latest versions generated by the server.  If a new version of the
stubs requires the client code to be changed before it will work, then
that new version of the server is _source backward incompatibile_.

Note that we are focusing on the compatibility of interfaces at the
protocol layer only.

## Examples

How important is backward compatibility?  How often are we likely to
deal with backward incompatible service versions?  Consider some
examples of each of the types of compatibility changes.


### Source Compatible Changes

The following changes to the service interface require no changes to
any clients.

Note that some of these may not hold true for java clients.

* Adding new methods or structs
* Adding a non-required field to a struct
* Changing a method return type from void to anything else
* Removing an exception from a method signature (ruby only)

### Binary Compatible Changes

The following changes to a service interface will require clients update their code when
updating to the latest service stubs.

* Removing a field from a struct
* Renaming a struct
* Adding an argument to a method with or without a default value
* Removing an argument from a method
* Changing the order of arguments in a method
* Changing the namespace 

### Binary Backward Incompatible Changes

The following changes require all clients to update their stubs to the latest versions generated.
Clients connecting with older versions of the stubs will get exceptions on their method invocations.

* Removing or renaming an interface method
* Removing a struct definition
* Removing a field from a struct
* Renaming a method
* Changing the declared type of a return value, or changing it to void
* Adding an exception to a method signature

## Running the Examples

You can see examples of the compatible changes listed above in a sample thrift service called the `AIS::Accounts`
service.  

There are three different versions of this service:

* 1.0.0: The first version
* 1.0.1: The second version with changes that are source compatible with clients written with the V1 stubs
* 1.1.0: The third version with additional changes that are binary compatible but not source compativle with V1 clients.

You can run a server daemon and a sample client with the [server runner](examples/server.rb) and 
[client runner](examples/client.rb) ruby files.  Both these scripts take a single command line argument, the
version of the interface to use: v1, v2, or v3.  To demonstrate binary compatibility, you can run each of the client
versions against the server v3 version.  

First install the required gems:

    gem install thrift
    gem install thin

Next generate the client stubs:

    cd examples
    ./gen.sh

Start up v3 of the server.  You can start up other versions of the service but the backward incompatibility scenarios
are all relative to the latest version of the server.

    ruby server.rb 1.1.0 &

Now run each version of the client.

    ruby client.rb 1.0.0
    ruby client.rb 1.0.1
    ruby client.rb 1.1.0

Furthermore you can study the changes required to the client in order to upgrade to v3 of the stubs.  You'll find these
changes described in the V3 module of [client.rb](examples/client.rb).

So what's the point of runnable examples?  If you have a question about the effect of any particular kind of change
on clients the best way to answer it is to actually try it out.  These examples all use ruby but you could verify
some of the assumptions using clients written in other languages.
