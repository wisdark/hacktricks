# Wireshark tricks

## Improve your Wireshark skills

### Tutorials

The following tutorials are amazing to learn some cool basic tricks:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### Analysed Information

#### Expert Information

Clicking on _**Analyze** --&gt; **Expert Information**_ you will have an **overview** of what is happening in the packets **analised**:

![](../../../.gitbook/assets/image%20%28571%29.png)

#### Resolved Addresses

Under _**Statistics --&gt; Resolved Addresses**_ you can find several **information** that was "**resolved**" by wireshark like port/transport to protocol, mac to manufacturer...  
This is interesting to know what is implicated in the communication.

![](../../../.gitbook/assets/image%20%28574%29.png)

#### Protocol Hierarchy

Under _**Statistics --&gt; Protocol Hierarchy**_ you can find the **protocols** **involved** in the communication and data about them.

![](../../../.gitbook/assets/image%20%28576%29.png)

#### Conversations

Under _**Statistics --&gt; Conversations**_ you can find a **summary of the conversations** in the communication and data about them.

![](../../../.gitbook/assets/image%20%28572%29.png)

#### **Endpoints**

Under _**Statistics --&gt; Endpoints**_ you can find a **summary of the endpoints** in the communication and data about each of them.

![](../../../.gitbook/assets/image%20%28575%29.png)

#### DNS info

Under _**Statistics --&gt; DNS**_ you can find statistics about the DNS request captured.

![](../../../.gitbook/assets/image%20%28577%29.png)

#### I/O Graph

Under _**Statistics --&gt; I/O Graph**_ you can find a **graph of the communication.**

![](../../../.gitbook/assets/image%20%28573%29.png)

### Filters

Here you can find wireshark filter depending on the protocol: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)  
Other interesting filters:

*  `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
  * HTTP and initial HTTPS traffic
*  `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
  * HTTP and initial HTTPS traffic + TCP SYN
*  `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
  * HTTP and initial HTTPS traffic + TCP SYN + DNS requests

### Search

If you want to **search** for **content** inside the **packets** of the sessions press _CTRL+f_  
You can add new layers to the main information bar _\(No., Time, Source...\)_ pressing _right bottom_ and _Edit Column_

Practice: [https://www.malware-traffic-analysis.net/](https://www.malware-traffic-analysis.net/)

## Identifying Domains

You can add a column that show the Host HTTP header:

![](../../../.gitbook/assets/image%20%28405%29.png)

And a column that add the Server name from an initiating HTTPS connection \(**ssl.handshake.type == 1**\):

![](../../../.gitbook/assets/image%20%28408%29.png)

## Identifying local hostnames

### From DHCP

In current Wireshark instead of `bootp` you need to search for `DHCP`

![](../../../.gitbook/assets/image%20%28409%29.png)

### From NBNS

![](../../../.gitbook/assets/image%20%28406%29.png)





## Decrypting TLS

### Decrypting https traffic with server private key

_edit&gt;preference&gt;protocol&gt;ssl&gt;_

![](../../../.gitbook/assets/image%20%28263%29.png)

Press _Edit_ and add all the data of the server and the private key \(_IP, Port, Protocol, Key file and password_\)

### Decrypting https traffic with symmetric session keys

It turns out that Firefox and Chrome both support logging the symmetric session key used to encrypt TLS traffic to a file. You can then point Wireshark at said file and presto! decrypted TLS traffic. More in: [https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/)  
To detect this search inside the environment for to variable `SSLKEYLOGFILE`

A file of shared keys will looks like this:

![](../../../.gitbook/assets/image%20%2862%29.png)

To import this in wireshark go to _edit&gt;preference&gt;protocol&gt;ssl&gt;_ and import it in \(Pre\)-Master-Secret log filename:

![](../../../.gitbook/assets/image%20%28191%29.png)

## ADB communication

Extract an APK from an ADB communication where the APK was sent:

```python
from scapy.all import *

pcap = rdpcap("final2.pcapng")

def rm_data(data):
    splitted = data.split(b"DATA")
    if len(splitted) == 1:
        return data
    else:
        return splitted[0]+splitted[1][4:]

all_bytes = b""
for pkt in pcap:
    if Raw in pkt:
        a = pkt[Raw]
        if b"WRTE" == bytes(a)[:4]:
            all_bytes += rm_data(bytes(a)[24:])
        else:
            all_bytes += rm_data(bytes(a))
print(all_bytes)

f = open('all_bytes.data', 'w+b')
f.write(all_bytes)
f.close()
```



