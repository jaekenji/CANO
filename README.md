## **Brief Explanation**

![cano2](https://github.com/user-attachments/assets/d9cda332-e786-4a54-a363-c235bf172981)

### **What is CANO?**

CANO, or Close Access Network Operations, is a method for gaining access to closed wireless networks by exploiting the WPA2 authentication process. It involves using a Raspberry Pi equipped with two Alpha Wi-Fi cards, one to monitor and another to transmit.

Raspberry Pi connects to a server via a cellular modem using WireGuard VPN, allowing an operator to manage the device. CANO encompasses both the technical tools deployed on the Raspberry Pi and the setup used by the end user to control and interact with it.

### **What is the Wirless Exploit?**

There isn't a single name for this exploit, but what is, is a **WPA2 Deauth + Handshake Capture Attack**

You send a deauthentication frame to forcibly disconnect a client from the access point. When the client reconnects, it performs a four-way handshake with the AP. You can capture this handshake, which contains a hash of the pre-shared key.

So the exploit isn’t breaking WPA2 directly—it's exploiting the unprotected nature of management frames and the fact that the handshake exposes a hash that can be attacked offline.
