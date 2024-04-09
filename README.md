# DNS_Exfil
DNS Exfiltration Setup on AWS

## Step 1: Purchase Domain Name on Route 53

## Step 2: Create EC2 instance to host DNS service and logging
Logging will contain DNS requests with encoded information

## Step 3: Note down default SOA and NS records, then remove them
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