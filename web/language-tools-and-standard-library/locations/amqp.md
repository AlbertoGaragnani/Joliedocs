## Amqp

In Jolie an amqp location implements an asynchronous communication between two services.

The communication is mediated by a queue defined in the RabbitMQ implementation of Amqp protocol.

Amqp location name in Jolie port definition is `amqp`.

An amqp location is an address expressed as a URI in the form `amqp://host:port?params`, where:

* `host` identifies the system running the RabbitMQ broker. It can be either a domain name or an IP address;
* `port` defines the port where the RabbitMQ broker is located.
* `params` are in the `key=value` form and are separated by the `&` character, params can be:

  * `exchange`: the name of the exchange used in the RabbitMQ communication, it acts as a mediator between the jolie service and the queue; 
  in order to use the default one it can be written in this form: `exchange=`. It can be omitted if you are defining an inputPort.
  * `queue`: the name of the queue.
  * `routingkey`: the key used to bind a message from the exchange to the queue; it has to be the same as queue name if you are using the default exchange.
  It can be omitted if you are defining an inputPort.

For the accurate definition of exchange and routingKey you can refer to this link: https://www.rabbitmq.com/getstarted.html

## Test

The easiest way to work with this amqp implementation is to have docker installed on your computer and pull the rabbitmq official image from 
https://hub.docker.com/ or just running the command `docker pull rabbitmq` from the command line.

Then run the image and check that the rabbitMQ service is correctly running on port `5672` (it's the default one).

Define two Jolie services by inserting the right string in `location` and remembering that by now only `OneWay` operations are implemented. Here is an example:

Client:

```jolie
include "console.iol"

type GreetRequest { name:string }
type GreetResponse { greeting:string }

interface GreeterAPI {
    OneWay: greet( GreetRequest )
}

service Greeter {

    outputPort GreeterInput {
        location: "amqp://localhost:5672?exchange=&queue=test&routingkey=test"
        protocol: sodep
        interfaces: GreeterAPI
    }

    main {
    	request.name = "juan"
        greet@GreeterInput( request )
    } 
}
```
Server:

```jolie
include "console.iol"

type GreetRequest { name:string }

interface GreeterAPI {
    OneWay: greet( GreetRequest )
}

service Greeter {
    execution: concurrent

    inputPort GreeterInput {
        location: "amqp://localhost:5672?queue=test"
        protocol: sodep
        interfaces: GreeterAPI
    }

    main {
        [greet( request )]{
            println@Console("Hello " + request.name)()
        }
    }
}
```
It is strongly recommended to use the `sodep` protocol, specially if you need to communicate with service defined in a non-Jolie language. 
By using this protocol the Amqp message will be casted to a JSON format such as the following:
`{"method":"greet","sodepAsync":"1.0","id":1,"params":{"name":"juan"},"resorucePath":"\/"}`
