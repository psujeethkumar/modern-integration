Context: Integration of remotely running MQ queue manager which is SSL enabeld.


Steps: 

1. Create an independent integration server
2. Create an policy project and an policy of type MQEndpoint in it.
3. Provide the remote MQ queue manager details like hostname, port, connection channel name, sl set to true, cipher spec : ANY TLS, security identity name
4. Provide the MQ connection credentials to the integration server through the mqsisetdbparams command.
