# Detecting Malicious Files Using VirusTotal

## **1. Enabling VirusTotal Integration in Wazuh**

This feature works on the **Wazuh Manager**, so I entered the manager container:

`sudo docker ps sudo docker exec -it <manager-container-id> bash`

Inside the container, I opened:

`nano /var/ossec/etc/ossec.conf`

I added (or enabled) the VirusTotal block:

`<virustotal>   <enabled>yes</enabled>   <api_key>YOUR_API_KEY_HERE</api_key>   <interval>10m</interval>   <timeout>60s</timeout> </virustotal>`

👉 I replaced `YOUR_API_KEY_HERE` with my actual VirusTotal API key.

Then I restarted the manager:

`/var/ossec/bin/ossec-control restart`

---

## **2. Ensuring Wazuh Monitors File Hashes**

VirusTotal works **only when a file hash is collected**.

So I made sure FIM/syscheck was enabled on the agent:

In my agent `/var/ossec/etc/ossec.conf`:

`<syscheck>   <enabled>yes</enabled>    <directories check_all="yes" realtime="yes">/home</directories>   <directories check_all="yes" realtime="yes">/tmp</directories> </syscheck>`

Restarted agent:

`sudo systemctl restart wazuh-agent`

Wazuh will now send file hashes (MD5/SHA1/SHA256) to the manager.

---

## **3. Testing with a Suspicious File**

To trigger VirusTotal scanning, I placed a suspicious file in a monitored directory:

Example:

`echo "fake malware test" > /tmp/test_malicious.txt`

Or downloaded an EICAR test file:

`curl -O https://secure.eicar.org/eicar.com.txt`

When the file was created/modified, FIM triggered events and Wazuh generated a hash.

---

## **4. VirusTotal Query Happens Automatically**

After the file hash was received, the Wazuh manager automatically:

- Sent the hash to VirusTotal
    
- Waited for a reply
    
- Attached the VirusTotal result as a new alert
    

This happens transparently — no manual API calls needed.

---

## **5. Viewing VirusTotal Results in Wazuh**

Inside **Wazuh Dashboard → Threat Hunting**, I filtered:

`rule.groups: virustotal`

or searched:

`virustotal`

Here I saw events like:

- `"positives": 0` → clean
    
- `"positives": 12` → malicious
    
- `"scan_date": ...`
    
- `"permalink": "https://www.virustotal.com/..."`
    
- `"resource": "sha256-hash"`
    

The detection rule usually looks like:

**"VirusTotal: Malicious file detected"**

This confirmed the file hash was matched against known malware signatures.

---

## **6. Key Things I Learned**

- VirusTotal doesn’t scan the file itself — it scans the **hash** from Wazuh.
    
- FIM (syscheck) must be enabled or Wazuh won't send hashes.
    
- VT integration only works on the **manager**, not the agent.
    
- You need your own API key for it to work.
    
- Results appear inside Wazuh automatically — no extra steps required.