# FAQ
**Q: Practice of application deployment under Mac system--when deploying mosquitto broker application, the status of user application broker shows that it has been deployed. When connecting to the edge device using mqtt.box, I encountered a Connection Error, what is the reason? **

** A: ** At present, the framework only supports edge devices under Linux systems. Because the host port mapping cannot work normally under Mac systems, it will cause mqtt.box connection failure. In addition, if you select manual activation during node provisioning, Mac systems also Unable to open the browser to activate.

