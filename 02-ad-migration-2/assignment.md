---
slug: ad-migration-2
id: 3b3rvhgwfj34
type: challenge
title: Extracting and Preparing the Data
teaser: Export DNS data, fix import errors, and re-upload to Infoblox.
notes:
- type: video
  url: https://videos.infoblox.com/watch/6GQfS55a86R1XrFEWAsQHc?
tabs:
- id: nsw68oxlqtqw
  title: Shell
  type: terminal
  hostname: shell
- id: mty7acntmpyz
  title: Editor
  type: code
  hostname: shell
  path: /root/infoblox-lab
- id: hqilqawkjpms
  title: AWS Console
  type: browser
  hostname: aws-console
- id: 1fambt8vpcyo
  title: Infoblox GM UI
  type: browser
  hostname: infoblox-gm-ui
- id: krl72v7xf5yz
  title: Domain Controller 1 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController1?username=instruqt&password=Passw0rd!
  port: 8080
- id: b48k60xlnucw
  title: Domain Controller 2 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController2?username=instruqt&password=Passw0rd!
  port: 8080
- id: fxj0ztneevjy
  title: Windows Client
  type: service
  hostname: rdpclient
  path: /#/client/c/WindowsClient?username=instruqt&password=Passw0rd!
  port: 8080
- id: txnrsxomtoun
  title: Lab Diagram
  type: browser
  hostname: lab
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: null
---
## 1) Introduction - Extracting and Preparing the Data
===

## Scenario Intro
With the environment validated, it’s time to pull the DNS data out of Active Directory. This is where you act like a field engineer — extracting, packaging, and inspecting the data before it ever touches the Infoblox system.

One of your datasets won’t import cleanly — and that’s on purpose. You’ll need to fix the file before continuing.

## Your Objectives
1.	Run the export script:
- From either DC1, run the provided export PowerShell script to extract the DNS data.
- The script will generate CSV files and automatically zip them for easy upload.
2.	Inspect the exported data:
- Unzip the export.
- Verify the CSV headers and record counts.
3.	Upload into Infoblox Migration Portal:
- Create a new migration job.
- Upload the dataset into Infoblox.
4.	Resolve import errors:
- One file will fail import due to formatting or policy restrictions.
- Use the provided Ansible playbook (or PowerShell cleanup script) to remove bad rows and adjust FQDN formatting (/ → -).
- Re-upload the corrected file.

## Outcome
By the end of this challenge, you’ll have successfully imported all valid DNS data into Infoblox, with hands-on experience cleaning and transforming real-world DNS exports.


## 2) Extracting the Data
===


🛠 Challenge: Extract the Data

With your Infoblox NIOS Grid up and running, it’s time to prepare the source data from your existing Microsoft AD DNS environment. This export will be used for validation and migration into NIOS.

📝 What you’ll do
- RDP into DC1
- Export all DNS zones and records from the AD-integrated DNS view
- Package them into a ZIP file for easy upload into the Infoblox Migration Portal

📂 Step 1: RDP to your Domain Controller 1

On the right-hand panel, click the DC1 tab to open the Windows desktop via RDP.

⚙ Step 2: Run the export script


1.	Open PowerShell from the Start menu.

![Screenshot 2025-08-14 at 09.58.46.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/a5b8aa2421235b0cccb9c93c68363533/assets/Screenshot%202025-08-14%20at%2009.58.46.png)

3.	Navigate to the export folder:

~~~
 cd  \
cd infoblox
 ~~~

![Screenshot 2025-08-14 at 10.03.05.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/cb1a7b593afdde5287595835e7500ff9/assets/Screenshot%202025-08-14%20at%2010.03.05.png)

4. Run the script:

~~~
.\extract-ms-dns.ps1 -DiscoverServers 1
~~~

Let the script finish — it may take a few moments depending on the number of zones and records.

![Screenshot 2025-08-14 at 10.04.30.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/cd4f917b71be5ecda68dbfb8bcd10e38/assets/Screenshot%202025-08-14%20at%2010.04.30.png)

This script will:
- Enumerate all zones in AD-integrated DNS
- Export them into right file format

When it’s done, you’ll see two export folders in C:\infoblox\ — one for each DC. Inside each folder are the extracted DNS zone files and metadata, ready for import into Infoblox.

⸻

📤 Step 3: Verify your export


🎥 Quick Demo: After the export script completes, watch this short video showing how to check the export folders and zip them properly.

[button label="Watch Export Verification Video"  background="#c2fbd7" color="green"](https://www.loom.com/share/5af30f9dd23548ffbc9c765d822fc72e?sid=da7af64a-2662-48e8-948d-f0431acca4d6)


To check: open **Windows File Explorer** ( you can type “Explorer” in the search bar or click the Explorer icon).
- Inside C:\infoblox, you should see two export folders (one per Domain Controller). Each folder contains the exported zone files and a .conf file.
- Inside, you should see all exported zone files and a zones

🗜 Zip the folders:

- Right-click each export folder → Compress to → ZIP File. You’ll end up with two ZIP files, one per DC, ready for upload.
- This makes them easier to upload during the import process.

![Screenshot 2025-08-14 at 10.10.36.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/f09eb4a2c578a29ed579b69d0c2e0b32/assets/Screenshot%202025-08-14%20at%2010.10.36.png)

## 3) 📦 DNS Data Processing with Infoblox Migration Tool
===

Now that you’ve extracted the DNS data from Microsoft AD, the next step is to reformat and refactor it into an Infoblox-friendly CSV structure ready for the final import.
We’ll use the Infoblox Migration Tool to handle the heavy lifting.

We give you a little help from an Infoblox friend below 🙂


You can either follow the **User Guide** or take a quick peek at the 🚀 **Video Tutorial** to make it clearer click on the button below:

[button label="🚀 Video Tutorial for Step 3" background="#90ee90" color="black"](https://www.loom.com/share/3e52dd93870245c89aea7b70874e2025?sid=35cfef5b-653e-40be-8d30-09be7b6cdc0f)




📍 Step 1 – Open the Migration Tool from DC1

1.	Open a browser ( Microsoft Edge ) and go to https://portal.infoblox.com.


💡 Tip: As shown in the screenshot, you can also use the pre-saved Infoblox CSP bookmark in the Favorites bar to get there quickly.

![Screenshot 2025-08-14 at 10.17.06.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/7bca89360833a0a2321ea958bb005c91/assets/Screenshot%202025-08-14%20at%2010.17.06.png)


2.	Log in with your username/email and password.

You’ll be prompted to accept:
- End-User License Agreement (EULA)
- Cookies consent

✅ Make sure you accept both before proceeding.

3.	Make sure you’re in the correct tenant that has DMS (Data Migration Service) access.
4.	Navigate to:
Configure > Administration > Migration.

![Screenshot 2025-08-14 at 10.21.59.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/9ebe1589fae132e1ff609f8dc1641ba6/assets/Screenshot%202025-08-14%20at%2010.21.59.png)

![Screenshot 2025-08-14 at 10.22.04.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/1468b38929e0e92d2c242af7360806bc/assets/Screenshot%202025-08-14%20at%2010.22.04.png)

✅ Success Check: You see the Migration Workspaces page.


📍 Step 2 – Create Your Workspace

1.	Click the blue “Create” button.
2.	Enter a Workspace Name: Demo and Description.

![Screenshot 2025-08-14 at 10.23.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c0e1cdec19e214e46cb615a09306aa16/assets/Screenshot%202025-08-14%20at%2010.23.48.png)

4.	Click Save & Close.
5.	From the list, click your new Workspace name to open it.

![Screenshot 2025-08-14 at 10.24.21.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/d9b371707ffb6a1be3326feaa775c9a9/assets/Screenshot%202025-08-14%20at%2010.24.21.png)

✅ Success Check: You are inside your new workspace.



📍 Step 3 – Create a DNS Migration Workload


1.	Click Create Migration Workload → DNS.

![Screenshot 2025-08-14 at 10.25.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/dd750c84e14d4f091f8f051eb1ef8248/assets/Screenshot%202025-08-14%20at%2010.25.40.png)

2.	In the wizard:

![Screenshot 2025-08-14 at 10.26.24.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/29868eb8d1baf740a0ad55a2a8fe2861/assets/Screenshot%202025-08-14%20at%2010.26.24.png)

- 🚨 Name your workload: DNS - 🚨 DNS Name is a MUST
- Select Microsoft  as the legacy system.
- Select Infoblox NIOS - NIOS 9.0.x -  as the target system.
- 👉 Enter Name Server Group = NSG (created earlier).


![Screenshot 2025-08-25 at 12.28.26.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/48f1cedd5eb06baab8b808bf9bc6cd07/assets/Screenshot%202025-08-25%20at%2012.28.26.png)

5.	Click Save & Next.

![Screenshot 2025-08-14 at 10.28.45.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/42c1dfb374169874e3adb63f01a823be/assets/Screenshot%202025-08-14%20at%2010.28.45.png)

6. DO NOT close the Infoblox window just open a new Windows Explorer
7.	On Domain Controller 1 (DC1), navigate to C:/infoblox folder using Windows Explorer
8.	Open the corresponding first folder (e.g., ec2amaz-xxxx.corp.infolab_ib_export) and search for the .conf file with the extension .conf (e.g., ec2amaz-xxxx.corp.infolab-named.conf).

> [!IMPORTANT]
> NOTE: Copy the exact .conf filename — we will need this later.

![Screenshot 2025-08-14 at 10.30.41.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/ee3c9691b51dd3c947c7d91bc04525a9/assets/Screenshot%202025-08-14%20at%2010.30.41.png)

![Screenshot 2025-08-14 at 10.30.56.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/e9e2f2d431ab625954495f4c1dd8596a/assets/Screenshot%202025-08-14%20at%2010.30.56.png)


9. Find the ZIP file that matches the first folder name in this location (e.g., ec2amaz-xxxx.corp.infolab_ib_export.zip).

10.	Now go back to the Infoblox Portal where we left off and continue with the wizard.

11. In the DNS Configuration file name field, paste the exact .conf file name you copied earlier (e.g., ec2amaz-9f9fqr4.corp.infolab-named).
- For DNS Record Data Source, click Select File and choose the ZIP file you created earlier (e.g., ec2amaz-9f9fqr4.corp.infolab_ib_export.zip).
- Leave DDNS Configuration File empty
- Click Save & Next to proceed.

![Screenshot 2025-08-14 at 10.37.49.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/715e7195f0e47c46b54ff4887a012b69/assets/Screenshot%202025-08-14%20at%2010.37.49.png)


12. Click Close on the next window.

![Screenshot 2025-08-14 at 10.39.58.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/7145b1eb0cc6851a66d8d4ccd4b92bd9/assets/Screenshot%202025-08-14%20at%2010.39.58.png)

After about 30 seconds, refresh the page — you will now see the uploaded file processed and ready for the next step.

> [!IMPORTANT]
> STATUS: Parsing Completed

![Screenshot 2025-08-14 at 10.42.21.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/6fd6ed959aeb5c2db6fc62ff0938acda/assets/Screenshot%202025-08-14%20at%2010.42.21.png)


✅ Success Check: You reach the status Parsing Completed.


📍 Step 5 – Generate CSV Files


1.	Inside your DNS Migration Workload, click on DNS.

![Screenshot 2025-08-14 at 10.48.12.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/87ce5557a0719cde27f786e3d53f79e4/assets/Screenshot%202025-08-14%20at%2010.48.12.png)

2.	Under Global DNS Configuration on next page select all items and Click on Set Import Option.

> [!NOTE]
> ⚠️ Note: If you can’t see the full portal page or data inside the GUI window, try expanding the Instruqt browser window.


![Screenshot 2025-08-14 at 10.49.10.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/2efcd2dca09e56ce00dd4b1ecc68eb2e/assets/Screenshot%202025-08-14%20at%2010.49.10.png)

3.	Select option Do not import from the drop-down menu and click Save&Close.

![Screenshot 2025-08-14 at 10.50.12.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c32d7f1bb9f1d217c4b5583df85a2150/assets/Screenshot%202025-08-14%20at%2010.50.12.png)

4.	Click on Generate CSV on the next page - Wait a few seconds for the process to complete — the Download CSV button will no longer be greyed out once it’s ready.

![Screenshot 2025-08-14 at 10.53.22.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/00fb77c7767c6f81ce38fdac66c1d1b1/assets/Screenshot%202025-08-14%20at%2010.53.22.png)

5. Click on Download CSV and wait for a few sec.

![Screenshot 2025-08-14 at 10.58.27.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/be0ca1d5a96a05c9adbc383a84c5e203/assets/Screenshot%202025-08-14%20at%2010.58.27.png)

6. The file will be saved in your Downloads folder — open File Explorer, navigate there, and right-click the .tar file to select Extract all

![Screenshot 2025-08-14 at 11.00.05.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/4a0b937d60570fc87b809857af62982c/assets/Screenshot%202025-08-14%20at%2011.00.05.png)

![Screenshot 2025-08-14 at 11.00.16.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c01f3dd7fae21505f2f15976d7fb6f92/assets/Screenshot%202025-08-14%20at%2011.00.16.png)

![Screenshot 2025-08-14 at 11.01.08.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/ad3f79e6a91f95f95bb446f0a6a0a068/assets/Screenshot%202025-08-14%20at%2011.01.08.png)

✅ Success Check: All CSV files are successfully extracted and ready for review before final import.


## 4) 🛠 DNS Data Formatting & Cleanup (Ansible Playbooks)
===

Now that you’ve successfully extracted the DNS data from the Migration Tool, it’s time to clean, format, and prepare it so it’s fully Infoblox-friendly for import.
We’ll use a purpose-built Ansible playbook nicknamed  file_adjustment.yml  to automate the cleanup and formatting process.

📍 Step 1 – Open the Linux Shell Tab

1.	In the left panel, select Linux Terminal (Shell).
2.	You’ll be running the commands from here.

📍 Step 2 – Navigate to the Playbook Directory

~~~run
cd /root/infoblox-lab/instruqt-aws-dc-lab-full/ansible/
ansible-galaxy collection install -r requirements.txt --force
~~~

![Screenshot 2025-08-14 at 11.08.29.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/e6cfbe8688d4cace4a750d4d0fcce1f4/assets/Screenshot%202025-08-14%20at%2011.08.29.png)

📍 Step 3 – Run the Playbook

~~~run
ansible-playbook -i inventory.ini file_adjustment.yml
~~~

![Screenshot 2025-08-14 at 11.12.23.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/163c4bf4d9fff695378831b3f2141cbe/assets/Screenshot%202025-08-14%20at%2011.12.23.png)


📍 Step 4 – Wait for the Playbook to Finish
The playbook will:
- Remove error-description columns.
- Drop records that violate Infoblox import policies.
- Save a cleaned version of the CSV in the same folder.


![Screenshot 2025-08-14 at 11.14.42.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/07d319f36b1f4efbfe0a3d1e8eb53acc/assets/Screenshot%202025-08-14%20at%2011.14.42.png)

✅ Success Check: Cleaned CSV is present in the Downloads/DNS_nios folder.The original file has been renamed to AuthZone.csv.bak for backup.


📍 Step 5 – Fetch the files using Ansible playbook and move it to Infoblox GM UI

~~~run
ansible-playbook -i inventory.ini fetch-infoblox.yml
~~~

![Screenshot 2025-08-14 at 11.21.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/a7f6d25df31d7f1d84dd93b3495628ab/assets/Screenshot%202025-08-14%20at%2011.21.40.png)

## 5) 🚀 Importing DNS Data into Infoblox NIOS
===


Now that our CSV files are ready, it’s time to bring them into the Infoblox Grid. We’ll follow the correct import sequence and verify the results. If we hit an error (we will on purpose for training), we’ll handle it like pros.

> [!NOTE]
> 🔐 Note: Since the Grid Manager session auto-logs out after idle timeouot, you’ll need to log back in with the provided lab credentials before starting the import process.


💡 Quick Video Guide: The video below walks you through importing the first AuthZone.csv. Click the button below to watch the video. Follow the same steps for all remaining files by following the guide below.

[button label="Video of importing  the zone files" background="#c2fbd7" color="green"](https://www.loom.com/share/70e7742940c2413897c47ddc7ba0f6e3?sid=1e6bc017-2a63-43fc-b9ed-eeb465567894)

📍 Step 1 – Back Up the Grid

⚠️ Do NOT perform this step in this lab.

In a Production environment, however, creating a Grid backup is mandatory before making any major configuration changes. Below are the steps you would normally follow in Production.

Go to Infoblox GM UI Tab in the left-hand part of the window and login if need to Grid Manager.

Before making major changes, always create a Grid backup:
1.	Navigate to Grid → Grid Manager → Toolbar → Backup.

![Screenshot 2025-08-14 at 11.24.31.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/758713d9db965661be19a5f4ffb8ea9c/assets/Screenshot%202025-08-14%20at%2011.24.31.png)

2.	Click Backup → Grid Backup → Manual Backup → select Backup to (My Computer) and click Backup.


✅ Success Check: Backup is available in the Backup Files list with today’s date.


📍 Step 2 – Import Authoritative Zones


1.	In Data Management → DNS → Toolbar, click CSV Import.

![Screenshot 2025-08-14 at 11.25.50.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/d24b61e4b266a311b594921b54d462ca/assets/Screenshot%202025-08-14%20at%2011.25.50.png)

2.	Select Add and click NEXT.

![Screenshot 2025-08-14 at 11.26.30.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c4079fa62d875d401288c62ebf899e2a/assets/Screenshot%202025-08-14%20at%2011.26.30.png)

3.	Select **Skip to the next row and continue** as the error-handling option. Then click Choose… to upload and import the AuthZone CSV file.

![Screenshot 2025-08-14 at 11.26.38.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/14ef29801d090c47f81f991b2c7b9071/assets/Screenshot%202025-08-14%20at%2011.26.38.png)

4. Follow the screenshots below to navigate to the correct folder and upload the AuthZone CSV.


- Click Choose Button (as shown in the previous step). This will open the Infoblox GM-UI file picker window.
- From here, navigate to:
- Home → user → DC1 → DNS_nios
- Inside this folder, you’ll find the cleaned CSV files generated by the Ansible cleanup playbook.
- Select AuthZone.csv and click Open to continue with the import.

⚠️ Note: The cleaned CSVs are always saved under /home/user/DC1/DNS_nios (or /home/user/DC2/DNS_nios if working with the second controller). Make sure you pick the right one for the step you’re performing.


Please follow the screenshots below.


![Screenshot 2025-08-14 at 11.26.47.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/002453554b677183fd0003e250f02d4c/assets/Screenshot%202025-08-14%20at%2011.26.47.png)

![Screenshot 2025-08-14 at 11.26.54.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/53dc8bc17f6834932a0dc9fe56e5e4f2/assets/Screenshot%202025-08-14%20at%2011.26.54.png)

![Screenshot 2025-08-14 at 11.27.04.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/7aa08b172ca856022ba48c9771699837/assets/Screenshot%202025-08-14%20at%2011.27.04.png)

5. Click Next.

![Screenshot 2025-08-14 at 11.27.11.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/33b838d518b87ee7496e3b5e6518831a/assets/Screenshot%202025-08-14%20at%2011.27.11.png)

6. Click Import and select YES.

![Screenshot 2025-08-14 at 11.28.56.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/d2112a7fd5825d2ca2702d0a509eea6f/assets/Screenshot%202025-08-14%20at%2011.28.56.png)

![Screenshot 2025-08-14 at 11.29.04.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c4edb8533dae65fbbc08f9d706ca81dd/assets/Screenshot%202025-08-14%20at%2011.29.04.png)

7. Wait for the Import to finish

![Screenshot 2025-08-14 at 11.29.24.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/a4d8eb4d3b9ac2790bd79093b6857490/assets/Screenshot%202025-08-14%20at%2011.29.24.png)



![Screenshot 2025-08-14 at 11.35.39.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/0c556053b38def8c25b4fd40dc8fdc60/assets/Screenshot%202025-08-14%20at%2011.35.39.png)

✅ Success Check: All authoritative zones are now listed under the DNS view.



📍 Step 3 – Import Remaining CSV Files (Same Procedure as Step 2)

Follow the exact same import process from Step 2 for each of the following files in this specific order:

1.	DelegatedZones.csv – ensures delegated zones are correctly created.
2.	ARecord.csv – imports A records.
3.	PtrRecord.csv – imports PTR mappings.
4.	CnameRecord.csv – imports CNAME aliases.
5.	SrvRecord.csv – imports SRV service mappings.

💡 Tip: After each import, quickly scan the NIOS UI to ensure records or zones appear as expected before moving on.


📍 Step 4 – Handling Errors During Import

If you see errors like this when you do importing of ARecord.csv. Click Download Errors.

![Screenshot 2025-08-14 at 11.45.49.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c469859d5148fba6771ae1ebba7c2321/assets/Screenshot%202025-08-14%20at%2011.45.49.png)


A) Inspect the error file on GM-UI

From the Shell Tab terminal:

### SSH to the GM-UI box
~~~run
ssh infoblox-gm-ui
~~~

> [!IMPORTANT]
> ⚠️ Note: This SSH session is to the GM-UI workstation/server, not an Infoblox appliance.


### Go to the Downloads folder and list files

~~~run
cd /home/user/Downloads
ls -la
~~~

### Peek at the error CSV (yours may be csv-error.X.csv)

~~~run
cat csv-error.4.csv
~~~

You should see lines like:

![Screenshot 2025-08-14 at 11.52.58.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/e2e9c50d088a8fa84ad1b2f208e9bb1c/assets/Screenshot%202025-08-14%20at%2011.52.58.png)


What’s wrong?
- fqdn contains / which is illegal → must be replaced with -
- A row says “Cannot add records to a zone that is not authoritative” → that line must be removed

Exit GM-UI when done inspecting (exit) or stay—Ansible will operate remotely anyway.

To exit GM-UI:

Make sure to type exit in the shell to properly log out of the GM UI.


B) Auto-fix the CSV with Ansible

Run this from Shell terminal (same dir as your playbooks):

~~~run
cd /root/infoblox-lab/instruqt-aws-dc-lab-full/ansible
ansible-playbook -i inventory.ini fix-csv-errors.yml
~~~

![Screenshot 2025-08-14 at 12.05.21.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/5eff521da7d9547b8d3b7a9d0e647c20/assets/Screenshot%202025-08-14%20at%2012.05.21.png)

Now that the error CSV file is fixed we will import it again following the import CSV procedure we did before.



📍 Step 5 – Re-Import Fixed CSV File

Now that the error CSV file is fixed, we will import it again following the same CSV import procedure we used earlier.


1. Go to the same Import CSV screen in the Infoblox GM-UI where the file originally failed. In Data Management → DNS → Toolbar, click CSV Import.

![Screenshot 2025-08-14 at 11.26.30.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c4079fa62d875d401288c62ebf899e2a/assets/Screenshot%202025-08-14%20at%2011.26.30.png)

![Screenshot 2025-08-14 at 11.26.38.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/14ef29801d090c47f81f991b2c7b9071/assets/Screenshot%202025-08-14%20at%2011.26.38.png)

3. Select the csv-error.<n>.fixed.csv file we just generated.

![Screenshot 2025-08-14 at 12.03.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/896e43f79dcdb92bcd9e3a1fd95bc4eb/assets/Screenshot%202025-08-14%20at%2012.03.48.png)


![Screenshot 2025-08-14 at 12.03.55.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/f080ad391e1e5df210d92d8c34a41421/assets/Screenshot%202025-08-14%20at%2012.03.55.png)

3.	Proceed through the wizard and complete the import.
4.	Confirm that previously failing records (e.g., FQDNs with / or delegation issues) are no longer present in the error report.

	![Screenshot 2025-08-14 at 12.04.19.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/64cfb5bf130e533bdf85df9767170067/assets/Screenshot%202025-08-14%20at%2012.04.19.png)

✅ Success Check: No errors are shown in the post-import CSV report.

⸻


## 6) 📦 Post-Import Parameter Adjustments on NIOS
===


After all CSV imports are successfully completed, we’ll fine-tune a few NIOS DNS service parameters to align with best practices and customer environment requirements.

📍 1. Allow Recursion

1. Navigate to Grid DNS Properties
	1.	In the NIOS UI, go to:
Data Management → DNS
	2.	In the toolbar on the right, click Grid DNS Properties (gear icon).Toggle Advance Mode and Select Queries on the left.

	![Screenshot 2025-08-20 at 11.27.01.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/ca6dffd4a7675dac20054a9934f4bbe2/assets/Screenshot%202025-08-20%20at%2011.27.01.png)

	![Screenshot 2025-08-20 at 11.27.12.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/cbdba8ceb0fcb0a62d2575b4393f1fcc/assets/Screenshot%202025-08-20%20at%2011.27.12.png)

	![Screenshot 2025-08-14 at 12.22.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/40925efb043955eed764d69bccb2801e/assets/Screenshot%202025-08-14%20at%2012.22.40.png)

	3.	Select Allow recursion and click Save & Close and click Yes on warning prompt.

	![Screenshot 2025-08-14 at 12.22.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/40925efb043955eed764d69bccb2801e/assets/Screenshot%202025-08-14%20at%2012.22.40.png)
	![Screenshot 2025-08-14 at 12.22.55.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/d9db127a51c725fab277f53de9a05ff7/assets/Screenshot%202025-08-14%20at%2012.22.55.png)

📍 2. Allow updates Part 1 - zone _msdcs.corp.infolab

💡 Tip: If the zone is hard to spot in the hierarchy, toggle to Flat View (top-right option). This makes zones easier to locate visually.

1.	Go to Data Management → DNS.
2.	Click on the zone and select Edit

![Screenshot 2025-08-14 at 12.23.47.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/3c2d27686880cc8c462f3b9ef3753707/assets/Screenshot%202025-08-14%20at%2012.23.47.png)

3.	Under Updates, select Override and Click Add button on the right select IPv4 Address.

![Screenshot 2025-08-14 at 12.25.11.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/6123d59bd50bec2e8ee27f499da7b75b/assets/Screenshot%202025-08-14%20at%2012.25.11.png)

4.	Add both DC1 (10.100.1.100) and DC2 (10.100.2.100).

![Screenshot 2025-08-14 at 12.26.05.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/7b856798dedf7000875e1e5f9214d812/assets/Screenshot%202025-08-14%20at%2012.26.05.png)

5.	Save & Close.


	📍 3. Allow updates Part 2 - zone  corp.infolab

1.	Go to Data Management → DNS.
2.	Click on the zone and select Edit

![Screenshot 2025-08-14 at 12.26.46.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/099c2a3f87035829bad3f7e08e64ddcf/assets/Screenshot%202025-08-14%20at%2012.26.46.png)

3.	Under Updates, select Override and Click Add button on the right select Any Address/Network.

![Screenshot 2025-08-14 at 12.27.19.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b4b67b3c91ef7acfbacf4d024452f6a1/assets/Screenshot%202025-08-14%20at%2012.27.19.png)

![Screenshot 2025-08-14 at 12.58.02.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/5f3da2ca1949db2f1fbded25cb336101/assets/Screenshot%202025-08-14%20at%2012.58.02.png)

4.	Save & Close.

📍 4. Allow Zone Transfers (for Microsoft DNS)

1.	Go to Data Management → DNS → Name Server Groups.

	![Screenshot 2025-08-14 at 13.00.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/00b79320f048b7c25c58f9708a0e9a85/assets/Screenshot%202025-08-14%20at%2013.00.48.png)

2.	Select NSG and click Edit.

	![Screenshot 2025-08-14 at 13.01.48.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/ee06ccd646ad8bb80b53bcb301b98e4a/assets/Screenshot%202025-08-14%20at%2013.01.48.png)

3.	Click Add→External Secondary

	![Screenshot 2025-08-14 at 13.02.21.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/f15b56970db79b577c477f778e2188e6/assets/Screenshot%202025-08-14%20at%2013.02.21.png)

4.	 Add below DCs following steps in the screenshots below.
	•	DC1: 10.100.1.100
	•	DC2: 10.100.2.100

![Screenshot 2025-08-14 at 13.04.01.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/5c23d3a99a6bf7cc6e8863c2b07db0bf/assets/Screenshot%202025-08-14%20at%2013.04.01.png)

![Screenshot 2025-08-14 at 13.04.09.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/019d4d616e73e0bc61bd5016f4fdbdfc/assets/Screenshot%202025-08-14%20at%2013.04.09.png)

![Screenshot 2025-08-14 at 13.05.05.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/0dafd5e02635c57d34248441edc98729/assets/Screenshot%202025-08-14%20at%2013.05.05.png)

6.	Click Save&Close.

🔄 Step – Restart Services to Apply Configuration
Once changes are saved, go to the top of the Infoblox Portal and click Restart Services. Confirm when prompted by clicking RESTART. This ensures all new DNS settings are applied across the Grid.

![Screenshot 2025-08-14 at 12.31.24.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b0585b278e10912a8ff9f07be375cbdb/assets/Screenshot%202025-08-14%20at%2012.31.24.png)

![Screenshot 2025-08-14 at 12.31.30.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/684eb7e0695f250ae8e63d8e3ea2c9d3/assets/Screenshot%202025-08-14%20at%2012.31.30.png)


🚀 Next Challenge – Going Live and Validating Service

Now that the Infoblox NIOS Grid is configured, CSVs are imported, and parameters are tuned, we’ll move to validating DNS query flows and performing a cutover simulation.


📍 Click NEXT in the lab to proceed to this challenge.

