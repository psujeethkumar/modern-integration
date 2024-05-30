1. Add the operator source
2. Install IBM Cloud Pak foundational services operator v4.4
3. Install IBM MQ operator v3.1
4. Install IBM App Connect operator v11.4
5. Create a project where in the ace and mq should be installed.
6. Create a secret object with credentials to the ibm containre registry to pull the images.

MQ Installation

1. Create a secret with mqwebsuer configuration
2. Create a ConfigMap object with required mqsc commands for the queue manager.
3. Create QueueManager object referring to the mqsc command related ConfigMap and mqwebuser related secret.
