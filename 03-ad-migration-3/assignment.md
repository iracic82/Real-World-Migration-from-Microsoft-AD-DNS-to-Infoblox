---
slug: ad-migration-3
id: xmrojctdol7w
type: challenge
title: Going Live and Validating Service
teaser: Switch DNS to Infoblox, test resolution, and confirm service works.
tabs:
- id: y7uww4mjmyck
  title: Shell
  type: terminal
  hostname: shell
- id: seinafrsydwe
  title: Editor
  type: code
  hostname: shell
  path: /root/infoblox-lab
- id: es3phiumd68s
  title: AWS Console
  type: browser
  hostname: aws-console
- id: utssdjltypm1
  title: Infoblox GM UI
  type: browser
  hostname: infoblox-gm-ui
- id: edcu58wgr1ag
  title: Domain Controller 1 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController1?username=instruqt&password=Passw0rd!
  port: 8080
- id: trnwec4up7xg
  title: Domain Controller 2 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController2?username=instruqt&password=Passw0rd!
  port: 8080
- id: pfthw91jxtkh
  title: Windows Client
  type: service
  hostname: rdpclient
  path: /#/client/c/WindowsClient?username=instruqt&password=Passw0rd!
  port: 8080
- id: ltq8f5nn3eda
  title: Lab Diagram
  type: browser
  hostname: lab
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: null
---
## Introduction - Going Live and Validating Service
===

## 🏁 Final Challenge – Go-Live & Validation


With your DNS data migrated and validated in Infoblox, it’s time to flip the switch. This is the moment the business has been waiting for — moving from Microsoft AD DNS to Infoblox in production.

You’ll reconfigure DNS on your DCs and client machines to point to the Infoblox Grid Master and then test resolution and browsing.


Now it’s time to prove it works and hand the keys to production.

📍 Step 1 – Validate DC1 against Infoblox

1. Go to the left part of the window in your lab and select the Domain Controller 1  - RDP (DC1) tab.
2.	RDP into DC1.
3.	In the Start Menu, type cmd and open the Command Prompt.

![Screenshot 2025-08-15 at 19.33.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/9cb7904eab8a06727f0b99abddbe2ce6/assets/Screenshot%202025-08-15%20at%2019.33.48.png)

5.	Type:

~~~
nslookup
~~~

and press Enter.

5.	Point queries to the Infoblox NIOS by typing:

~~~
server 10.100.2.11
~~~

and press Enter.

6.	Test public resolution:

~~~
google.com
~~~

and press Enter.

You should see a valid public IP response.

~~~
Server:  [10.100.2.11]
Address: 10.100.2.11

Non-authoritative answer:
Name:    google.com
Addresses:  2a00:1450:4001:801::200e
            216.58.212.142
~~~


7.	Test internal resolution (against an imported zone record):

~~~
ws1562.blox.corp
~~~

Expected response:

~~~
Name: ws1562.blox.corp
Address: 10.192.54.143
~~~


✅ Success Check: Both tests return correct results via Infoblox, confirming DC1 is now resolving through the Grid Master.



📍 Step 2 – Validate DC1 against MS AD


1.	On the Domain Controller 1 (DC1) RDP session, open Command Prompt:
•	In Start Menu, type cmd, and hit Enter.

![Screenshot 2025-08-15 at 19.33.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/9cb7904eab8a06727f0b99abddbe2ce6/assets/Screenshot%202025-08-15%20at%2019.33.48.png)

2.	In the terminal, start nslookup and set DC1 (10.100.1.100) as the DNS server:

~~~
nslookup
server 10.100.1.100
~~~

![Screenshot 2025-08-15 at 19.36.28.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/645a8fa0d02fe7113e6719e4f0416cdf/assets/Screenshot%202025-08-15%20at%2019.36.28.png)

3.	Test resolution for both a public domain and an internal record:

- google.com
- ws1562.blox.corp


Expected Output:

![Screenshot 2025-08-15 at 19.36.56.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/1f8502a18cf148085c08630fb6881883/assets/Screenshot%202025-08-15%20at%2019.36.56.png)

![Screenshot 2025-08-15 at 19.37.13.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/32b1054864205b0c612fbc3b3175068f/assets/Screenshot%202025-08-15%20at%2019.37.13.png)


✅ Success Check:
•	Public query resolves (google.com).
•	Internal query resolves to 10.192.54.143 from the Infoblox zone.


📍 Step 3 – Validate DC2 against MS AD

1.	On Domain Controller 2 (DC2), open Command Prompt.
2.	Start nslookup and set DC1 (10.100.1.100) as the DNS server:

~~~
nslookup
server 10.100.1.100
~~~

3.	Run the same validation queries:

~~~
google.com
ws1562.blox.corp
~~~

Expected Output: (same as DC1)

~~~
Server:  [10.100.1.100]
Address: 10.100.1.100

Non-authoritative answer:
Name:    google.com
Addresses:  2a00:1450:4001:801::200e
            216.58.212.142

Name:    ws1562.blox.corp
Address: 10.192.54.143
~~~


✅ Success Check:

- Both public and internal lookups resolve identically on DC1 and DC2.
- Infoblox NIOS and MS AD DNS data are fully in sync.
- Data flows are validated end-to-end.


💡 Next: We can now proceed with the Go-Live script to re-point the DCs and clients to Infoblox as the primary DNS source.


## 🚀 Go-Live – Cutover to Infoblox
==

With validation complete, it’s time to make Infoblox NIOS the primary DNS source in production.

### 📍 Step 1 – Run the Go-Live Script on DC1


1.	Log in to DC1 via RDP.

- In Start Menu, type PowerShell, and hit Enter.

![Screenshot 2025-08-15 at 19.46.56.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/9748fed52ae4564b0774f5dec98c0dcf/assets/Screenshot%202025-08-15%20at%2019.46.56.png)

2.	Navigate to:
~~~
C:\infoblox
~~~

![Screenshot 2025-08-15 at 19.46.34.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/548277ebaf84d9b9ee7f48a232a27903/assets/Screenshot%202025-08-15%20at%2019.46.34.png)

3.	Run:

~~~
.\ad-dc-convert-go-live.cmd
~~~

📝 What This Step Does

In this stage, the script prepares your Active Directory DNS server for migration by carefully backing up its configuration and refreshing how DNS zones are handled. Here’s what happens under the hood:
- Safely pauses DNS – The DNS service is temporarily stopped so changes can be applied without risk of corruption.
- Takes a full backup – Both the DNS zones and global server settings are exported and also copied into a backup area in the registry. This ensures that nothing is lost and you can roll back if needed.
- Cleans out existing zones – The old zone entries are cleared from the registry. (Don’t worry—your records still live in Active Directory.)
- Switches the boot method – The DNS server is reconfigured to load its settings directly from the registry instead of AD DS. This makes the migration process more predictable and easier to control.
- Applies new forwarders – Updated DNS forwarders are added so the server knows where to send queries it can’t resolve locally.
- Restarts DNS and Netlogon – Finally, the services are restarted so the changes take effect and Active Directory registration refreshes cleanly.

![Screenshot 2025-08-15 at 19.46.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/0f4f716e88be2a98054ecfedbc7a5481/assets/Screenshot%202025-08-15%20at%2019.46.40.png)

4.	Once the script completes, open NIC Adapter Settings on DC1.

5.	Open Network Adapter Settings

- On the bottom-right Windows taskbar, right-click the network/monitor icon.
- Click Network & Internet settings.
- Click on Ethernet
- Edit DNS Setting


	![Screenshot 2025-08-15 at 19.28.38.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/4d39e23aae6e201b5192f977e74377d8/assets/Screenshot%202025-08-15%20at%2019.28.38.png)

	![Screenshot 2025-08-15 at 19.28.53.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/12467b7768d54b89be90f1b3a4b0e47f/assets/Screenshot%202025-08-15%20at%2019.28.53.png)

	![Screenshot 2025-08-15 at 19.29.29.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b2d671fb5149d862229e40bfc31a59e5/assets/Screenshot%202025-08-15%20at%2019.29.29.png)

6.	Change the Preferred DNS server to value below and remove Alternate DNS settings

~~~
10.100.2.11
~~~

![Screenshot 2025-08-15 at 19.31.47.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/98f29cabb428fe751c24688983c8797b/assets/Screenshot%202025-08-15%20at%2019.31.47.png)

![Screenshot 2025-08-15 at 19.32.06.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/d0eefffb5fbdc51804316adb26699634/assets/Screenshot%202025-08-15%20at%2019.32.06.png)

6.	Open Command Prompt and run:

~~~
nslookup
google.com
ws1562.blox.corp
~~~

> [!NOTE]
> No need to set the server manually — queries should now resolve through Infoblox.


### 📍 Step 2 – Run the Go-Live Script on DC2


1.	Repeat the exact same steps as Step 1 but for DC2.
2.	Update its Preferred DNS server to:


~~~
10.100.2.11
~~~


3.	Validate lookups:

~~~
nslookup
google.com
ws1562.blox.corp
~~~

4. Optionally test a new A record created in Infoblox to ensure updates are working live.

~~~
nslookup ws1039.ws.blox.corp
~~~

Expected result:
~~~
Name:    ws1039.ws.blox.corp
Address: 10.42.101.229
~~~


### 📍 Step 3 – Final Client Validation

With both DC1 and DC2 now serving DNS via Infoblox, the last step is to validate from an end-user perspective.

In production, this DNS cutover would typically be rolled out using Group Policy Objects (GPO), endpoint management tools, or DHCP scope updates to ensure the change propagates seamlessly to all clients.

For our lab, we’ll make the change manually on the Windows Client.


Steps:

1.	Open Network Adapter Settings
- On the bottom-right Windows taskbar, right-click the network/monitor icon.
- Click Network & Internet settings.
- Click on Ethernet
- Edit DNS Setting


	![Screenshot 2025-08-15 at 19.28.38.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/4d39e23aae6e201b5192f977e74377d8/assets/Screenshot%202025-08-15%20at%2019.28.38.png)

	![Screenshot 2025-08-15 at 19.28.53.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/12467b7768d54b89be90f1b3a4b0e47f/assets/Screenshot%202025-08-15%20at%2019.28.53.png)

	![Screenshot 2025-08-15 at 19.29.29.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b2d671fb5149d862229e40bfc31a59e5/assets/Screenshot%202025-08-15%20at%2019.29.29.png)


2.	Change the Preferred DNS server to:

~~~
10.100.2.11
~~~

![Screenshot 2025-08-15 at 19.29.53.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/1d53772e0f821422c214641cf5e95b31/assets/Screenshot%202025-08-15%20at%2019.29.53.png)

3.	Open Command Prompt and run validation queries:

~~~
nslookup google.com
nslookup ws1562.blox.corp
nslookup ws1039.ws.blox.corp
~~~

4.	Confirm that:
- Public queries (e.g., google.com) resolve without errors.
- Internal corporate records (e.g., ws1562.blox.corp, ws1039.ws.blox.corp) resolve to the expected IPs.


✅ Success Criteria:
- All queries resolve correctly, confirming that Infoblox is now the authoritative DNS source for both internal and external name resolution.
- The migration from Microsoft AD DNS to Infoblox is complete, data is in sync, and the cutover is successful.

## 🎯 Migration Complete – From Microsoft AD DNS to Infoblox
==

We started this journey in the first challenge with raw Microsoft AD DNS data buried inside the Domain Controllers, walking through extraction, transformation, and import into Infoblox.
Along the way, we:
- Extracted DNS data directly from DC1 and DC2.
- Reformatted and cleaned the datasets to make them Infoblox-ready.
- Imported Authoritative Zones, Delegated Zones, and Resource Records.
- Adjusted NIOS parameters for recursion, updates, and zone transfers.
- Validated resolution paths between Infoblox and Microsoft AD.
- Performed a controlled Go-Live cutover.

Today, your Infoblox Grid Master is handling all DNS resolution—both internal and external—across the environment.
The data is in sync, query flows are healthy, and your clients are resolving from the new authoritative source.

As the DNS lead, your next operational steps could include:
- Monitoring Infoblox’s reporting for query patterns and anomalies.
- Decommissioning the DNS role on Microsoft AD.
- Automating future changes with Infoblox APIs and integration into CI/CD pipelines.

From this point forward, your DNS infrastructure is modernized, centrally managed, and built for scale. The migration was not just a cutover—it was an upgrade to a more resilient, secure, and automated DNS platform.

Welcome to the new era of DNS. 🚀
