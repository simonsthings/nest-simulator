---
layout: index
---

[« Back to the index](index)

<hr>

# Writing synapse models

NEST has a very flexible system to allow users to write their own
synapse types. Synapses in NEST can either have static parameters or
apply some dynamics on them. Each connection needs to have at least
the following parameters:

* The connection delay
* The connection weight
* The target node of the connection
* The receiver port, which identifies the connection on the postsynaptic side 

The source node of a connection is implicitly stored by the position
in the data structure that is used internally. These parameters are
implemented in the StaticConnection synapse type, which can be used as
a base class for more advanced synapse types.  Writing a Connection
object

Writing a synapse type is basically very simple. You can directly
derive your new connection type from StaticConnection, which provides
all mechanisms to register and send a connection. A synapse type
consist of two files, a header and an implementation. Skeletons for
both of them are shown shown in the following listings:

`NESTSRC/models/my_connection.h`

    #ifndef MY_CONNECTION
    #define MY_CONNECTION
    
    #include "static_connection.h"
    #include "generic_connector.h"
    
    namespace nest
    {
      class MyConnection:public StaticConnection
      {
        public:
          MyConnection () {}
          MyConnection (const MyConnection &) {}
          ~MyConnection () {}
    
          update_dynamics ();
          void send (Event & e, double_t t_lastspike, const CommonSynapseProperties & cp);
      };
    
      inline void MyConnection::send (Event & e, double_t t_lastspike, const CommonSynapseProperties &)
      {
        update_dynamics();
    
        e.set_receiver(*target_);
        e.set_weight(weight_);
        e.set_delay(delay_);
        e.set_rport(rport_);
        e();
      }
    } // namespace nest
    
    #endif /* #ifndef MY_CONNECTION */

The first thing we do is include the header files of our base class,
StaticConnection. It already defines funtions for registering the
connection with the ConnectionManager of NEST, for storing the
mandatory parameters weight and delay and functions to set and
retrieve these parameters from within the SLI interpreter.

`NESTSRC/models/my_connection.cpp`

    #include "my_connection.h"
    
    void nest::MyConnection::update_dynamics ()
    {
      /* Do fancy stuff with weights here! */
    }

To apply some (activity dependent) dynamics on the weight of the
connection you simply have to override the method send(). It is the
one that is called each time an event flows over the
connection. Except for the call to update_dynamics() in which the
synaptic weight is calculated, the function MyConnection::send() is a
copy of the implementation from StaticConnection. It fills in the rest
of the parameters of the event and sends the event to the target.

## Registering the new synapse type

After your files are written, you have to add their names to the
libmodelmodule_la_SOURCES variable in NESTSRC/models/Makefile.am to
have it be compiled and linked to NEST.

To make the synapse type available inside of NEST scripts, you have to
include and register it with the module. Add the following line to the
begin of NESTSRC/models/modelmodule.cpp:

    #include "my_connection.h"

And the following line to the end of the file:

    register_prototype_connection<MyConnection>(net_, "my_synapse");
