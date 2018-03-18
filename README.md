# broker-adapter
Adapter to allow out-of-process broker implementation, receiving and sending raw XML through fifos in the local filesystem

## R Example
Example: The r-broker directory contains a brief working demo called forkDemo.R. It works within RStudio. Here's the process for running it:

* Create two fifos in the broker-adapter directory. The rest of this assumes you name them broker-input and broker-output
* In RStudio, source forkDemo.R
* In RStudio, change your working directory with setwd() to the broker-adapter directory.
* If you don't have a boot record, start the Power TAC server and generate a boot record.
* In the Power TAC server, set up a game using the boot record, and authorize the broker Rbroker to log in. Specify `pause.props` as the Server-config. Start the sim.
* In RStudio, run the command runBroker("broker-output", "broker-input")

You should see all the incoming xml messages streamed to the R console.

## General idea

You can create any kind of adapter by implementing the `IpcAdapter` interface and notifying the broker core to use it via a CLI parameter `--ipc-adapter`.
The broker-core will look for Spring beans that implement the `IpcAdapter` interface and use the first one it finds. There should be only one in the broker's classpath.

The broker implementation must reimplement certain features that are otherwise handled automatically in the Java version of the broker. One such example is the handling of the `broker-accept` message, which looks like

```
<broker-accept prefix="2" key="dlop5b" serverTime="1521046957413"/>
``` 

The broker core takes care of prepending the key to outgoing messages, but the implementation needs to use the prefix multiplied by 100000000 to calculate unique ID values for domain types sent to the server, such as tariff-specs and orders. This is essential; otherwise the server will discard your messages. To see how this is done in Java, see the [`IdGenerator`](https://github.com/powertac/powertac-core/blob/master/common/src/main/java/org/powertac/common/IdGenerator.java) class. The serverTime is the time in milliseconds UTC at which the first timeslot starts. This time advances by one timeslot-duration each time the server sends out the TimeslotUpdate message.

You can see examples of all the xml message types by starting the server and running a test game with the sample-broker (use the 2week-game.props config in server-distribution to avoid wasting too much time). The trace log from the sample-broker will contain every incoming and outgoing xml message.
