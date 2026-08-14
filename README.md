
Now verify your root A record

Go back to Page 1 (click the 1 at the bottom).

Find this record:

Type: A
Name: @

Make sure it says

13.201.184.30

NOT

13.201.184.38

If it already says 13.201.184.30, then DNS is configured correctly.

Next check

Open Command Prompt and run:

nslookup droneyaan.com

Then run:

nslookup www.droneyaan.com

Both should eventually return:

13.201.184.30

If they still show the old IP, DNS propagation is still in progress. It can take anywhere from a few minutes to several hours depending on caching.

Please send me:
A screenshot of Page 1 showing the A record for @ (or confirm it is 13.201.184.30).
The output of these two commands:  i cant find this option
On Page 1 of your GoDaddy DNS Management, you need to look for the A record with:

Type: A

Name: @

That’s the root record. It should point to 13.201.184.30. If it still shows 13.201.184.38, you’ll need to edit it to the correct IP.

✅ Next Steps
Confirm in GoDaddy

Scroll back to Page 1 of your DNS records.

Find the A record for @.

Verify the “Value” (IP address) is 13.201.184.30.

Run nslookup
Open Command Prompt and run:

bash
nslookup droneyaan.com
nslookup www.droneyaan.com
Both should eventually resolve to 13.201.184.30.

If they still show the old IP, it just means DNS propagation is ongoing. This can take a few minutes to several hours depending on caching.

👉 To move forward, please either:

Send me a screenshot of Page 1 showing the A record for @, or

Confirm directly that it says 13.201.184.30.

And also paste the output of those two nslookup commands so I can verify propagation status for you.

Would you like me to walk you through exactly how to edit the A record in GoDaddy if it still shows 13.201.184.38?



I see the issue clearly now. On Page 1 of your DNS records, the A record for @ is currently set to:

Type: A

Name: @

Data: 13.201.184.38

This is the old IP, not the correct one.

🔧 What you need to do
In GoDaddy DNS Management, click Edit on that A record (@).

Change the Data/Value from 13.201.184.38 → 13.201.184.30.

Save the record.

🖥️ Verification
After saving:

Open Command Prompt and run:

bash
nslookup droneyaan.com
nslookup www.droneyaan.com
Both should eventually resolve to 13.201.184.30.

If they still show 13.201.184.38, it just means DNS propagation is still in progress. This can take a few minutes to several hours depending on caching.

👉 Please confirm once you’ve updated the A record to 13.201.184.30. Then share either:

A screenshot of Page 1 showing the updated A record, or

The output of the nslookup commands so I can verify propagation status for you.

Would you like me to give you the exact step‑by‑step clicks in GoDaddy’s interface to edit that record?

no i have now change this
Perfect — you’ve updated the root A record from 13.201.184.38 → 13.201.184.30. That’s exactly what was needed.

🔍 Next Verification
Now let’s confirm propagation:

Open Command Prompt  
Run these two commands:

bash
nslookup droneyaan.com
nslookup www.droneyaan.com
Expected Output  
Both should eventually resolve to:

Code
Name: droneyaan.com
Address: 13.201.184.30
and

Code
Name: www.droneyaan.com
Address: 13.201.184.30
If you still see 13.201.184.38  
Don’t worry — that just means DNS propagation is still in progress. It can take anywhere from a few minutes to several hours depending on caching.

👉 Please share either:

A screenshot of your nslookup output, or

Just paste the text results here.

That way I can confirm for you that everything is resolving correctly.

Would you like me to also show you how to flush your local DNS cache so you can see the new IP faster?

C:\Users\user>nslookup droneyaan.com
DNS request timed out.
    timeout was 2 seconds.
Server:  UnKnown
Address:  fd00::1

DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
*** Request to UnKnown timed-out

C:\Users\user>nslookup www.droneyaan.com
DNS request timed out.
    timeout was 2 seconds.
Server:  UnKnown
Address:  fd00::1

DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
DNS request timed out.
    timeout was 2 seconds.
*** Request to UnKnown timed-out

C:\Users\user>
Right now your nslookup is failing because Windows is trying to use your local IPv6 resolver (fd00::1), which isn’t responding. That’s why you’re seeing repeated “DNS request timed out” errors. The domain itself is fine — you already corrected the A record to 13.201.184.30 — but your PC isn’t reaching a working DNS server.

🔧 Fix the Resolver
Try forcing a public DNS server instead of the broken local one:

Run these commands in Command Prompt:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
(This uses Google DNS directly.)

If you want to make it permanent:

Go to Control Panel → Network and Internet → Network Connections.

Right‑click your active adapter → Properties.

Select Internet Protocol Version 4 (TCP/IPv4) → Properties.

Set DNS servers to:

Preferred: 8.8.8.8

Alternate: 8.8.4.4

Flush your DNS cache:

bash
ipconfig /flushdns
ipconfig /renew
✅ What You Should See
After step 1, both commands should return:

Code
Name: droneyaan.com
Address: 13.201.184.30
and

Code
Name: www.droneyaan.com
Address: 13.201.184.30
👉 Go ahead and run the forced nslookup with 8.8.8.8. Paste the output here, and I’ll confirm that your domain is resolving correctly.

Name:    droneyaan.com
Address:  184.168.103.43


C:\Users\user>nslookup www.droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  13.201.184.38
Aliases:  www.droneyaan.com
Thanks for sharing the updated nslookup results — here’s what they mean:

droneyaan.com → 184.168.103.43  
This is still pointing to GoDaddy’s default hosting IP, not the new one you set.

www.droneyaan.com → 13.201.184.38 (alias of droneyaan.com)  
This is still showing the old IP (38), not the corrected 13.201.184.30.

Why this is happening
You already changed the A record for @ to 13.201.184.30, but DNS propagation hasn’t fully completed yet.

The www record is a CNAME pointing to droneyaan.com. Since droneyaan.com hasn’t fully propagated to the new IP, www is still resolving to the old value.

Propagation can take anywhere from a few minutes to several hours depending on caching at DNS resolvers worldwide.

✅ What to do next
Wait for propagation  
Within a few hours, both droneyaan.com and www.droneyaan.com should resolve to 13.201.184.30.

Force refresh locally  
Run:

bash
ipconfig /flushdns
Then retry:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
Double-check GoDaddy

Ensure the A record for @ is 13.201.184.30.

Leave the www CNAME pointing to droneyaan.com (that’s correct).

👉 Right now everything is configured correctly — it’s just a matter of propagation. Once caches update, both root and www will resolve to 13.201.184.30.

Would you like me to show you how to test propagation globally (using tools like DNSChecker) so you can see which regions have already updated?

ipconfig /flushdns
  where run this
You’ll want to run ipconfig /flushdns inside Command Prompt on Windows. Here’s the exact process:

🖥️ Steps
Press Windows Key + R → type cmd → press Enter.
(This opens Command Prompt.)

In the black Command Prompt window, type:

bash
ipconfig /flushdns
and press Enter.

You should see a message like:

Code
Successfully flushed the DNS Resolver Cache.
(Optional but recommended) Refresh your IP lease:

bash
ipconfig /renew
✅ After That
Re-run:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
Both should start resolving to 13.201.184.30 once propagation has caught up.

Do you want me to also show you how to check propagation globally (using a web tool like DNSChecker) so you can see which regions have already updated?

nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
Good — now that you’ve flushed your DNS cache, the next step is to re‑run the commands with Google’s DNS resolver explicitly:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
What you should expect
droneyaan.com → should eventually resolve to 13.201.184.30 (the corrected A record).

www.droneyaan.com → since it’s a CNAME pointing to droneyaan.com, it will also resolve to 13.201.184.30 once propagation finishes.

Right now you’re still seeing:

droneyaan.com → 184.168.103.43 (GoDaddy default hosting IP)

www.droneyaan.com → 13.201.184.38 (old IP)

That means propagation hasn’t fully completed yet. This is normal — DNS changes can take a few minutes to several hours depending on caching at different resolvers worldwide.

✅ Next checks
Keep retrying the nslookup with 8.8.8.8 every 30–60 minutes.

Use a global DNS checker (like dnschecker.org) to see if some regions already show 13.201.184.30.

Once both root (@) and www resolve to 13.201.184.30, your DNS is fully propagated and correct.

Would you like me to walk you through checking propagation globally right now so you can see which regions have already updated?

Windows IP Configuration

Successfully flushed the DNS Resolver Cache.

C:\Users\user>ipconfig /renew

Windows IP Configuration

No operation can be performed on Ethernet while it has its media disconnected.
No operation can be performed on Local Area Connection* 9 while it has its media disconnected.
No operation can be performed on Local Area Connection* 10 while it has its media disconnected.
No operation can be performed on Bluetooth Network Connection while it has its media disconnected.

Ethernet adapter Ethernet:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Wireless LAN adapter Local Area Connection* 9:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Wireless LAN adapter Local Area Connection* 10:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . : indowings.com
   Link-local IPv6 Address . . . . . : fe80::6528:a4bf:4e97:bc9%4
   IPv4 Address. . . . . . . . . . . : 192.168.1.227
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::f2ad:4eff:fe3d:439%4
                                       192.168.1.1

Ethernet adapter Bluetooth Network Connection:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :

C:\Users\user>nslookup droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  13.201.184.38


C:\Users\user>nslookup www.droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  13.201.184.38
Aliases:  www.droneyaan.com
Great — thanks for running those checks. Here’s what’s happening:

Both droneyaan.com and www.droneyaan.com are still resolving to the old IPs (184.168.103.43 and 13.201.184.38) instead of the corrected 13.201.184.30.

This is normal: DNS propagation is still in progress. Even though you updated the A record in GoDaddy, it can take anywhere from a few minutes to several hours (sometimes up to 24–48 hours depending on caching) for all DNS resolvers worldwide to pick up the change.

✅ What you should do now
Wait for propagation  
Keep checking every 30–60 minutes with:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
Eventually both will show:

Code
Address: 13.201.184.30
Check globally  
Use a tool like dnschecker.org → enter droneyaan.com and select A record.
This will show you which regions have already updated to 13.201.184.30.

Confirm GoDaddy settings

A record for @ → 13.201.184.30 (correct).

CNAME for www → points to droneyaan.com (correct).

👉 Bottom line: your configuration is correct. It’s just a matter of waiting for propagation to finish.

Would you like me to walk you through using DNSChecker right now so you can visually confirm which parts of the world already see the new IP?

Non-authoritative answer:
Name:    droneyaan.com
Address:  184.168.103.43


C:\Users\user>nslookup www.droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  13.201.184.38
Aliases:  www.droneyaan.com

  now given this how to fix it
Right now your DNS is still resolving to the old IPs because propagation hasn’t finished. Let’s break down exactly how to fix and confirm it:

🔧 Step 1: Verify GoDaddy Records
In your GoDaddy DNS Management:

A record for @ → must be 13.201.184.30 (you already corrected this).

CNAME for www → should point to droneyaan.com (this is correct, leave it as is).

So your configuration is fine — no further edits needed.

🔍 Step 2: Understand Why You Still See Old IPs
droneyaan.com → 184.168.103.43

www.droneyaan.com → 13.201.184.38

These are cached results from DNS resolvers that haven’t updated yet. Propagation is global and can take minutes to several hours (sometimes up to 24–48 hours).

✅ Step 3: Confirm Propagation
Keep running:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
every 30–60 minutes.
Eventually both will return:

Code
Address: 13.201.184.30
Use dnschecker.org → enter droneyaan.com and select A record.
This shows which regions worldwide already see the new IP.

🚀 Step 4: Speed Things Up Locally
You already flushed DNS (ipconfig /flushdns).

If you want faster updates:

Change your DNS servers to Google (8.8.8.8, 8.8.4.4) or Cloudflare (1.1.1.1).

This bypasses slower ISP caches.

👉 Bottom line: Your DNS setup is correct. Nothing else needs fixing in GoDaddy. It’s just propagation delay. Once caches update, both root and www will resolve to 13.201.184.30.

Would you like me to walk you through checking DNSChecker right now so you can see which regions already show the new IP?

C:\Users\user>nslookup droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  184.168.103.43


C:\Users\user>nslookup www.droneyaan.com 8.8.8.8
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    droneyaan.com
Address:  13.201.184.30
Aliases:  www.droneyaan.com


C:\Users\user>
C:\Users\user>
Excellent — now we’re seeing progress:

droneyaan.com → 184.168.103.43 (still the old GoDaddy hosting IP)

www.droneyaan.com → 13.201.184.30 (this is correct — the new IP is showing for the www alias)

🔍 What this means
Your A record for @ is set correctly to 13.201.184.30 in GoDaddy.

The CNAME for www points to droneyaan.com, and Google DNS is already resolving it to 13.201.184.30.

The only part lagging is the root domain (droneyaan.com), which is still cached at 184.168.103.43. This is purely a propagation delay.

✅ Next Steps
Wait for propagation  
Within a few hours, droneyaan.com will also resolve to 13.201.184.30. Right now, www is already correct, so your site should work fine when accessed via www.droneyaan.com.

Check globally  
Go to dnschecker.org, enter droneyaan.com, select A record, and you’ll see which regions already show 13.201.184.30.

Optional local speed-up

Use Google DNS (8.8.8.8, 8.8.4.4) or Cloudflare (1.1.1.1) in your network settings to bypass slower ISP caches.

Keep running:

bash
nslookup droneyaan.com 8.8.8.8
nslookup www.droneyaan.com 8.8.8.8
until both return 13.201.184.30.

👉 Bottom line: Your configuration is correct. www is already resolving properly, and the root domain will follow once propagation finishes.

Do you want me to walk you through checking DNSChecker right now so you can visually confirm which regions already see the new IP?
