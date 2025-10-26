---
slug: ad-migration-1
id: bog0tjaiy8l2
type: challenge
title: Getting the Lay of the Land
teaser: Check existing zones on DC1/DC2, verify AWS resources, and log into Infoblox.
notes:
- type: video
  url: https://cdn.instruqt.com/assets/instruqt/2024-036%20Instruqt%20101_Version%203.0.mp4
tabs:
- id: rdpveuxw6zg1
  title: Shell
  type: terminal
  hostname: shell
- id: apeakwvvweg0
  title: Editor
  type: code
  hostname: shell
  path: /root/infoblox-lab
- id: brytahpvab3z
  title: AWS Console
  type: browser
  hostname: aws-console
- id: wjyrwkhdtrtk
  title: Infoblox GM UI
  type: browser
  hostname: infoblox-gm-ui
- id: ecy7rnzn1tft
  title: Infoblox Portal
  type: browser
  hostname: infoblox
- id: ofn3t2xhwivt
  title: Domain Controller 1 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController1?username=instruqt&password=Passw0rd!
  port: 8080
- id: kboc9d0ovncj
  title: Domain Controller 2 - RDP
  type: service
  hostname: rdpclient
  path: /#/client/c/DomainController2?username=instruqt&password=Passw0rd!
  port: 8080
- id: al6htmfyagq4
  title: Windows Client
  type: service
  hostname: rdpclient
  path: /#/client/c/WindowsClient?username=instruqt&password=Passw0rd!
  port: 8080
- id: tpmhz73pb8pp
  title: Lab Diagram
  type: browser
  hostname: lab
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: null
---
## 1) Introduction - Migrating from Microsoft AD DNS to Infoblox NIOS – A Real-World Scenario
===

#  ⚡ Pre-Intro: Consultant’s Lens

Here you’ll experience a migration the same way an Infoblox Certified Consultant would.
The proprietary tools in this lab are available only through Infoblox Professional Services and Partner Professional Services, but the takeaway is clear:

💡 Infoblox experts bring the process, discipline, and speed that make customer migrations simple.


___

## Track Introduction

You are the DNS Lead for a global ACME enterprise that’s going all-in on a cloud-first strategy.
Identity services are moving to Microsoft Entra ID (Azure AD), which means the on-premises Active Directory Domain Controllers — currently hosting DNS  — are on the chopping block.

The mandate from leadership is clear:

“Migrate our DNS services off Microsoft AD, keep everything online, and future-proof our infrastructure — no downtime, no excuses.”

## Business Use Case

This migration isn’t just about replacing a service — it’s about modernizing how critical network services are delivered:
- Cost Reduction – Eliminate expensive, high-maintenance Windows Server DNS/DHCP roles.
- Operational Risk Mitigation – Deploy DNS/DHCP in a highly available, cloud-native architecture.
- Centralized Control & Visibility – Manage DNS globally with policy-based enforcement.
- Scalability & Agility – Avoid hardware refresh cycles and easily extend services to new sites or clouds.

Your Mission in this Track

As the DNS Lead, you are responsible for executing the migration end-to-end. In this live hybrid lab, DC1 and DC2 are still running, but by the time you’re done, they’ll be retired — replaced by Infoblox NIOS as the new authoritative DNS platform.

You will:
1. Configure the Infoblox NIOS Grid Master to be migration-ready.
2. Export DNS data from AD, review it, and package it for import.
3. Create and run a migration job in the Infoblox Migration Portal — handling a deliberately broken dataset and fixing it before import.
4. Validate DNS resolution from DC1, DC2, and the Client.
5. Execute go-live cutover scripts on DC1 and DC2.
6. Test resolution and browsing from the Client after switching DNS to NIOS.

By the end of this track, you’ll have delivered a zero-downtime migration, resolved real-world data quality issues, and proven that Infoblox NIOS can take over seamlessly — ready for the cloud era.

Let’s see how Infoblox can help you take DNS from legacy on-prem to modern, cloud-ready infrastructure.

Before any migration begins, you need full situational awareness.
As the DNS Lead, you’ve just taken ownership of the project — your first move is to review the current DNS environment, confirm the migration targets, and validate access to all involved systems.

Think of this as a recon mission: no changes yet, just verifying the battlefield.

🔍 Tip: Check the Lab Diagram for a visual simulation of the setup before proceeding.


## 2) Login to your cloud account consoles
===
🔐 Access Instructions

Using the credentials below, log in to the AWS and Azure Web Consoles.

---
# AWS Credentials ☁️

🔐 Logging In to the AWS Console

👉 First, open the “AWS Console” tab on the left-hand side of your Instruqt lab environment. This will launch the AWS login page in a new browser panel.

![Screenshot 2025-07-12 at 11.23.29.png](https://play.instruqt.com/assets/tracks/atmmwsclkofd/86d80bec0e3af0161dbb62f6e26e2626/assets/Screenshot%202025-07-12%20at%2011.23.29.png)

Then follow these steps:
1.	Select “IAM Account”
On the login screen, choose IAM Account (not root).

![Screenshot 2025-07-12 at 11.23.29.png](https://play.instruqt.com/assets/tracks/atmmwsclkofd/86d80bec0e3af0161dbb62f6e26e2626/assets/Screenshot%202025-07-12%20at%2011.23.29.png)

2.	Enter the AWS Account ID, AWS IAM username, and password by copying and pasting the values from the section below.

📝 Note: Avoid the root account login — this lab is configured for IAM users only.

**AWS Account ID**
```
[[ Instruqt-Var key="INSTRUQT_AWS_ACCOUNT_INFOBLOX_DEMO_ACCOUNT_ID" hostname="shell" ]]
```

**AWS IAM Username**
```
[[ Instruqt-Var key="INSTRUQT_AWS_ACCOUNT_INFOBLOX_DEMO_USERNAME" hostname="shell" ]]
```

**AWS Password**
```
[[ Instruqt-Var key="INSTRUQT_AWS_ACCOUNT_INFOBLOX_DEMO_PASSWORD" hostname="shell" ]]
```
---

## 3) Validate AD/DNS Baseline Before Migration
===

✅ Verify that the environment is healthy and that DNS data is in place before migration begins.


### 💡 Lab Pre-Provisioning Recap

Your lab environment has already been prepped:

- Two Windows Domain Controllers (DC1 & DC2) deployed and configured.
- DC1 promoted as the forest root with AD-integrated DNS.
- DC2 joined and promoted as an additional domain controller with DNS.
- Active Directory & DNS roles installed and configured.
- DNS zones and records imported to Microsoft AD.
- Basic forwarders and replication set up between the two DCs.

### 🔍 Your task now:
Log in to DC1 and DC2 via RDP, open DNS Manager, and explore the imported zones and records to get familiar with the current setup. This is the baseline you’ll migrate to Infoblox.


### 🔍 Accessing DNS Manager on DC1 and DC2


1. In the Tabs panel on the left of your lab interface, click Domain Controller 1 – RDP to open the RDP session for DC1.
2. Once the Windows desktop loads, click Start Menu → type DNS → click DNS.

![Screenshot 2025-08-14 at 09.46.51.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/a355aaa8c8391dc9a62b37d524780027/assets/Screenshot%202025-08-14%20at%2009.46.51.png)

4. In DNS Manager, expand your server node to review Forward Lookup Zones and verify imported zones/records.

![Screenshot 2025-08-14 at 09.47.15.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c8f4a8edc76c065ac1e5854f7ac07dde/assets/Screenshot%202025-08-14%20at%2009.47.15.png)


🎥 Quick Demo: Want to see this validation (and maybe run it yourself)  in action from the client perspective? Click on the green button below.


[button label="Quick Validation Peek" background="#c2fbd7" color="green"](https://www.loom.com/share/d63efd783b564e24ae84a959e84d4f34?sid=61136739-2051-4d07-a03a-cc8386623d11)



5. Repeat the same process for Domain Controller 2 – RDP to confirm replication and consistency.

💡 Tip: This quick check ensures your lab environment matches the pre-provisioned state before you start the migration.


## 4) 🛠 Checking and Setting up Infoblox NIOS
===


Before we migrate, we need to ensure the Infoblox NIOS appliance is ready to serve as our new authoritative DNS platform.


### Step 1 – Access the Infoblox Grid Manager

1.📍 Navigate to the NIOS Grid Manager UI

From the left-hand side panel, click on the “**Infoblox GM UI**” tab to open the application interface.

Alternatively, you can click the button below to launch the GM_UI directly in your current browser tab:

⬇️

[button label="GM UI"  background="#c2fbd7" color="green"](https://[[ Instruqt-Var key="SANDBOX_ID" hostname="shell" ]]-infoblox.iracictechguru.com)

2.	If you get a security warning, choose Advanced → Accept the Risk and Continue.

![Screenshot 2025-08-13 at 15.22.50.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/f5c8f12826c3b423e6a2a202588d8e78/assets/Screenshot%202025-08-13%20at%2015.22.50.png)

4.	Log in with:

Username:

```
admin
```

Password:

```
Proba123!
```

Note: Before starting the configuration, make sure the EULA (End User License Agreement) has been accepted. If the EULA is not accepted, the setup wizard will not allow you to proceed.

![Screenshot 2025-08-13 at 15.22.13.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/93f8628431a9b3dea0cd1e9dcc288842/assets/Screenshot%202025-08-13%20at%2015.22.13.png)

![Screenshot 2025-08-19 at 20.30.03.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/8ff1c729abb3de290d9daddbd0d02e82/assets/Screenshot%202025-08-19%20at%2020.30.03.png)


Note on License Warnings


You may see yellow banner warnings about:
- expiring trial/temporary licenses,
- subscription renewals, or
- CP license documentation.


![Screenshot 2025-08-19 at 20.35.12.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/431ad667d98513674bf743bd40d628e5/assets/Screenshot%202025-08-19%20at%2020.35.12.png)

👉 For lab/demo environments, you can safely ignore these license warnings during setup.
They won’t block Grid initialization or core feature enablement.
Please click Close button on each one of the warnings to free up the screen space.

⚠️ In production, make sure you replace temporary licenses with valid subscription licenses before expiration. Otherwise, services will stop.

⸻

Step 2 – Configure the Grid

After your first login, the Grid Setup Wizard should appear automatically.

When the Grid Setup Wizard first opens, make sure you select:
- Configure a Grid Master ✅


![Screenshot 2025-08-13 at 15.23.09.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/e80d2616ecb9e95e20dff734f2c324ca/assets/Screenshot%202025-08-13%20at%2015.23.09.png)


If it doesn’t, simply follow the first the steps below to launch it manually.

1.	Navigate to Grid → Grid Manager.
2.	In the right toolbar, Click on Grid Properties and select Setup Wizard.

![Screenshot 2025-08-13 at 15.25.20.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/582df4133c680152854f5e53a70e1894/assets/Screenshot%202025-08-13%20at%2015.25.20.png)

3.	Fill in:
- Grid Name: Infoblox
- Shared Secret: Proba123!
- Host Name: infoblox.localdomain
- Network Type: IPv4


![Screenshot 2025-08-14 at 09.22.30.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/6d5e49a2f29be0d4aa940649efc9f992/assets/Screenshot%202025-08-14%20at%2009.22.30.png)

4.	Confirm the LAN1 IP (10.100.2.11) and gateway settings.

![Screenshot 2025-08-14 at 09.22.40.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/dfad0520528252a451842d3360b2854f/assets/Screenshot%202025-08-14%20at%2009.22.40.png)

5.	Set up admin password

Password:

~~~
Proba123!
~~~

![Screenshot 2025-08-14 at 09.25.44.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/2233f173f514b20de698ff930c6618ff/assets/Screenshot%202025-08-14%20at%2009.25.44.png)

6.	Complete the wizard and wait for the Grid Master to initialize.

![Screenshot 2025-08-14 at 09.25.51.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/145076c392c8daac2027dacc740088ee/assets/Screenshot%202025-08-14%20at%2009.25.51.png)

![Screenshot 2025-08-14 at 09.25.58.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/64cd2386fc714a51fecdd4502f3bc629/assets/Screenshot%202025-08-14%20at%2009.25.58.png)

⸻

Step 3 – Enable DNS Service

1.	Still under Grid → Grid Manager, click the DNS Tab.

![Screenshot 2025-08-14 at 09.29.13.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/39ffcf297e02bfe7129918d3b3f70aae/assets/Screenshot%202025-08-14%20at%2009.29.13.png)

3.	Check the box next to infoblox.localdomain.
4.	Click on Start button.▶️

![Screenshot 2025-08-14 at 09.29.25.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/c3d5699c63bc62547ae4b7c8196fc147/assets/Screenshot%202025-08-14%20at%2009.29.25.png)

5. Click YES on the pop-up window.

![Screenshot 2025-08-14 at 09.29.34.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/2c911b25490f187cb09b0116d43c6666/assets/Screenshot%202025-08-14%20at%2009.29.34.png)

✅ Tip: After you click Yes, give it a few seconds. Refresh the page, and you should see the DNS service come alive and ready to roll.

⸻

Step 4 – Configure the NSG (Name Server Group)

1.	Go to Data Management → DNS → Name Server Groups.

![Screenshot 2025-08-14 at 09.34.35.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/5aa77e4693b3b8ddf90fd0f175440282/assets/Screenshot%202025-08-14%20at%2009.34.35.png)

3.	Create a new Authoratitive Name Server Group called NSG.

![Screenshot 2025-08-14 at 09.35.43.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b4b5f5572060405b519896470569220f/assets/Screenshot%202025-08-14%20at%2009.35.43.png)


5.	Click the ➕  plus icon on the right and choose Add Grid Primary.

![Screenshot 2025-08-14 at 09.35.50.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/a58a52e21a25f0a4a936df886f3d8116/assets/Screenshot%202025-08-14%20at%2009.35.50.png)

6. Then click the  SELECT button.
After a moment, you’ll see infoblox.localdomain appear in the list.

![Screenshot 2025-08-14 at 09.36.03.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/4bfd21bf482173ebd70d16df06469f9d/assets/Screenshot%202025-08-14%20at%2009.36.03.png)

7. Click the ADD button

![Screenshot 2025-08-14 at 09.36.10.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/b546a7a1bd385e29dcc8338d60e37f05/assets/Screenshot%202025-08-14%20at%2009.36.10.png)

8. Click Next and Save&Close

![Screenshot 2025-08-14 at 09.36.18.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/fdab7f41d8f42263d242ac50baf41807/assets/Screenshot%202025-08-14%20at%2009.36.18.png)

![Screenshot 2025-08-14 at 09.36.27.png](https://play.instruqt.com/assets/tracks/e6jecxl5nvew/202ba12af7bb0d9389ab5df85c386ab6/assets/Screenshot%202025-08-14%20at%2009.36.27.png)

⸻

✅ Now the Grid Primary is set — you’re ready to start importing DNS data into Infoblox NIOS.

## 5) Create an Admin User for your Infoblox Portal tenant
===

> [!IMPORTANT]
> Note: Use your Business Email for User Creation

Your user account and sandbox tenant have already been created. The next step is to input your password and activate your account.

> [!IMPORTANT]
> If you’ve never accessed the Infoblox Portal before using the email address you used to start this lab, please follow the steps outlined below in Section 1 to activate your account.
However, if you’ve previously engaged with the Infoblox Portal using the same email, your account is likely already active and associated with the correct tenant. In that case, you can skip Section 1, and optionally skip Section 2, unless you’ve forgotten your password and need to reset it.

### Section 1

1. Please check the inbox of the email account you used to register for the Infoblox Lab.
2. You will receive an email with the subject “Infoblox User Account Activation”. Open this email and click on the “Activate Account” button to proceed.

![Screenshot 2025-04-01 at 11.15.44.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/93d7e021e28c20d105ca34aace35d149/assets/Screenshot%202025-04-01%20at%2011.15.44.png)

4. A new browser window or tab will open, prompting you to create a new password. Please enter your desired password to complete the setup.

![Screenshot 2025-04-01 at 11.19.01.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/0fd5315d47a283fce641de81b289ac64/assets/Screenshot%202025-04-01%20at%2011.19.01.png)

5. Once you’ve set your password, close the newly opened tab or window. We’ll be logging in through the Instruqt tab labeled "Infoblox Portal".
6. In the Instruqt tab labeled "Infoblox Portal", log in using the credentials you set up in the previous steps.
7. After logging in, please mark the first window as shown in the example below. This confirms that you have successfully accessed the Infoblox Portal.

![Screenshot 2025-04-01 at 11.01.03.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/a082b5c7276e1c2e27eaaf7bee832089/assets/Screenshot%202025-04-01%20at%2011.01.03.png)

7. In the upper-left corner of the Infoblox Portal, click on the drop-down menu. Use the “Find Account” field to search for your sandbox by entering the "Sandbox-ID" shown below. It is important that you select your specific Sandbox-ID from the list and click on it to proceed.

**Your Sandbox ID**
```
[[ Instruqt-Var key="Sandbox_ID" hostname="shell" ]]
```

---


![Screenshot 2025-07-18 at 09.16.24.png](https://play.instruqt.com/assets/tracks/atmmwsclkofd/09e41ec1cf57a6f2cbe5d2c47721a26b/assets/Screenshot%202025-07-18%20at%2009.16.24.png)

---

### Section 2 ⚠️ (Troubleshooting) Help! I Forgot My Password


✅ Existing User

1.	Go to Infoblox Portal tab on the left.
2.	Log in using your existing email address and password.
3.	Once authenticated, the lab tenant will be automatically added to your list of available tenants (you’ll see it in the top-right tenant switcher).

![Screenshot 2025-07-18 at 09.16.24.png](https://play.instruqt.com/assets/tracks/atmmwsclkofd/09e41ec1cf57a6f2cbe5d2c47721a26b/assets/Screenshot%202025-07-18%20at%2009.16.24.png)

In order to RESET the password follow the steps below:

1. Please navigate to the Infoblox Portal page by clicking on the Infoblox Portal tab within the Instruqt lab environment. This will direct you to the appropriate login interface.
2. Once you’re on the Infoblox Portal page, click on “Need Assistance” located at the bottom of the login form.

![Screenshot 2025-04-01 at 10.52.47.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/0bacd7dc4193c2770df54b65c7eceb3a/assets/Screenshot%202025-04-01%20at%2010.52.47.png)

3. After clicking on “Need Assistance”, select “Forgot Password” from the available options to initiate the password reset process.

![Screenshot 2025-04-01 at 10.52.57.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/3971464a872c428ee8879f3d7287b201/assets/Screenshot%202025-04-01%20at%2010.52.57.png)

4. You will receive an email with the subject “Account Password Reset.” Open this email and click on the “Reset Password” button to proceed with setting a new password.

![Screenshot 2025-04-01 at 11.42.21.png](https://play.instruqt.com/assets/tracks/ywozzymyekgv/263657baa43cefce4e8fc2062d2371f1/assets/Screenshot%202025-04-01%20at%2011.42.21.png)

5. Once you’ve set up your password, please return to "Section 1" of the instructions and continue from Step 4 onward.




## Time for the Next Challenge

Now we've inspected the playing field its game time. Click **NEXT**!
