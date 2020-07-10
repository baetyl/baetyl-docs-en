# FAQ
#### Q: Practice of application deployment under Mac system, when deploying mosquitto broker application, the status of user application broker shows that it has been deployed. When connecting to the edge device using mqtt.box, I encountered a Connection Error, what is the reason? 

**A:**  Because the host port mapping cannot work normally under Mac systems, it will cause mqtt.box connection failure.

#### Q: When SN files are used to activate edge devices in batches, the deployment status is always only baetyl-init running. What is the reason? 

**A:**  There is a problem with the file mapping in the system directory under the Mac system. You can only choose to place the SN file in the user directory. Please modify the SN file path and deploy again.

