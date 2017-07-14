# Configure Zeppelin

Nodes that will run Zeppelin notebooks will need to have language support installed, including:
* R
* Java / Scala

# Install Zeppelin

Zeppelein requires
* Spark History Server
* Spark Thrift Server
* Livy Server

### Install Zeppelin as a service

Using Ambari, Services -> Actions button -> Install Zeppelin

### Install Spark (not Spark2 yet)

Using Ambari Services -> Actions button -> Install Spark

### Install Livy Server

Once Spark has been installed, you need to install the Livy Server.
Curiously, it is not available from the Services menu.

To install the Livy Server, go to the Hosts menu, select the Host on which Zeppelin is installed.
Click on the Components Add+ button and select the Livy Server.

![Livy_Server](/doc/img/LivyServerInstall.png?raw=true "Livy Server Install")

### Configure Livy Server

The default livy server timeout is 1 hour, and when it times our both livy and zepplin need to be restarted, 
so set the timeout to be 24 hours (actually 86400000 milliseconds) in Ambari 
via the livy.server.session.timeout property in the "Advanced livy-conf" section of the Spark service.

