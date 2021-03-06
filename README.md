
# RedvsBlue
Summary: Project where the role of a pentester and a SOC analyst are explored


## Network Topology

![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/Topology/Project2.jpg "Network Topology")

### Network
Address Range: 192.168.1.0/24
Netmask: 255.255.255.0
Gateway: 192.168.1.1

### Machines
| Name     | IPv4          | OS             |
|----------|---------------|----------------|
| Capstone | 192.168.1.105 | Ubuntu 18.04.1 |
| Kali     | 192.168.1.90  | Kali Linux     |
| Elk      | 192.168.1.100 | Ubuntu 18.04.4 |


# Red Team

## Vulnerability Assessment

| Vulnerability            | Description                                                                                  | Impact                                                                                                                                              |
|--------------------------|----------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| Directory Listing        | The server exposes the directory structure of the webserver and anyone can access the files. | Attackers can read the file contents and gain more information about the organization. Sensitive data can be stolen.                                |
| Sensitive Data Exposure  | When critical data lands on unauthorized hands.                                              | Exposing sensitive data to unauthorized parties means that it can be stolen.                                                                        |
| Weak Passwords           | Short passwords or common passwords.                                                         | Weak passwords can be easily brute-forced.                                                                                                          |
| Unrestricted File Upload | Uploaded files represent a significant risk to applications, specially executables.          | The first step in many attacks is to get some code to the system to be attacked. Then the attack only needs to find a way to get the code executed. |



### Exploitation: Directory Listing
1. Tools and Processes:
   - nmap to discover port 80 is open
   - Firefox to navigate to the Capstone machine.
   
2. Achievements:
   - We can see the Web Server file structure and read the contents of the files.
 
3. Screenshots:
   - Run: nmap -sV 192.168.1.105
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/1_1_nmap.PNG "nmap service scan")
   
   - Use Firefox and navigate to: http://192.168.1.105
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/2_1_directory_listing.PNG "Directory Listing")

### Exploitation: Sensitive Data Exposure
1. Tools and Processes:
   - Firefox web browser to read the contents of the files found in the Capstone WebServer
   
2. Achievements:
   - Thanks to error messages we found:
      - A hidden directory: /company_folders/secret_folder
      - A username candidate: ashton 
 
3. Screenshots:
   - Information about the secret directory:
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/3_1_file1txt.PNG "secret directory")
   
   - Information about possible username:
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/3_2_secret_folder_auth.PNG "username")

### Exploitation: Weak Passwords #1
1. Tools and Processes:
   - After finding out that ???ashton??? is a valid username, and that there is a secret folder that needs his credentials, we used Hydra and the rockyou dictionary to brute force the password and gain access to that directory.
   
2. Achievements:
   - We were able to find the password in approximately two minutes:
      - Username: ashton
      - Password: leopoldo 
 
3. Screenshots:
   - Run Hydra in our Kali Machine: ```hydra -l ashton -P rockyou.txt -s 80 -f -vV 192.168.1.105 http-get  /company_folders/secret_folder```
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/4_1_ashton_hydra.PNG "Hydra results")
  
### Exploitation: Weak Passwords #2
1. Tools and Processes:
   - Using the credentials for ???ashton???, we had access to the company???s secret_folder and found a file containing a hashed password for username ryan.
   - We used the Crack Station website to attempt to retrieve the password.
   
2. Achievements:
   - Crack Station was able to break the hashed password in less than five minutes:
      - Username: ryan
      - Password: linux4u 
 
3. Screenshots:
   - crackstation.net:
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/5_1_crackstation.PNG "Crackstation")
  
### Exploitation: Unrestricted File Upload
1. Tools and Processes:
   - With the information gathered  previously we obtained the instructions on how to upload files into the WebDAV application.
   - Use File Explorer in our Kali machine to upload a PHP payload.
   - Use Metasploit to set up the listener process.
   - Firefox to navigate to the PHP payload.
   
2. Achievements:
   - Reverse shell and access to the flag. 
 
3. Screenshots:
   - Upload Payload:
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/6_1_shareddirectorycontents.PNG "PHP Payload")

   - Flag:
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/6_2_flag.PNG "Flag")



# Blue Team

## Analysis using Kibana

### Analysis: Identifying the Port Scan

1. Attack occurred between 18:05and 18:30 (Jun/12/2021)
2. Smaller spike between 18:10 and 18:15 is a Port Scan.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/1_1_PortScan.PNG "Port Scan")
3. Source of the scan is: 192.168.1.90 (Kali Machine)
4. We can see that it is a port scan because there's constant traffic to various destination ports even when those ports are not in use and there's some icmp packets too.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/1_2_PortScan_Destination_Ports.PNG "Destination Ports")
   
   
### Analysis: Finding the Request for the Hidden Directory

1. The requests for the hidden directory occurred between 18:15 and 18:25 (Jun/12/2021).
2. A total of 16,284 requests were made.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/2_1_HiddenDir_Requests.PNG "Hidden Directory Requests")
3. The file requested from this directory is: connect_to_corp_server, which contains the information about how to connect to the WebDAV application.

   
### Analysis: Uncovering the Brute Force Attack

1. The number of requests made during the brute force attack were: 16,284.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/3_1_BF_HydraUserAgent.PNG "Brute Force Attack")
2. The number of requests made before the attacker discovered the password were: 16,282.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/4_1_BF_hits_before_correct_pswd.PNG "Brute Force Attack unsuccessful requests")


### Analysis: Finding the WebDAV Connection

1. There were 80 requests made to WebDAV during the attack.
   ![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/BlueTeam/5_1_webdav_requests.PNG "WebDAV requests")
2. The files requested were mainly:
   - passwd.dav (containing ryan???s hashed password), and 
   - meterpreter.php (payload)



## Proposed Alarms and Mitigation Strategies

### Mitigation: Blocking the Port Scan
1. Alarms:
   - If there are high amounts of requests from the same IP address to ports not in use, then blacklist that IP Address.
   - We would recommend when more than 20 requests are made in less than 1 minute.
2. System Hardening:
   - Setup a Firewall and add the following rules: 
         - Allow ping Echo-Request outbound and Echo-Reply messages inbound.
         - Allow traceroute TTL-Exceeded and Port-Unreachable messages inbound.
   - Schedule an audit of the system every Tuesday to patch its software and close any ports not in use.


### Mitigation: Finding the Request for the Hidden Directory
1. Alarms:
   - Alert when the number of failed attempts to access this directory exceeds 60 attempts per minute. 
   - Alert when there???s 5 attempts per minute to access the hidden directory from IP addresses not in our whitelist.
2. System Hardening:
   - This is a Web Server that needs to be accessed by possible customers, with the exception of the secret folder (which is not public). Whitelist the IP addresses that need access to the secret folder. Add the following to apache???s .htaccess file:
       ```json
           <Limit GET POST PUT>
                Order deny,allow
                Deny from all
                Allow from <IP address here>
                Allow from <IP address here>
                Allow from <IP address here>
            </Limit>
        ```

   - Disable Apache Directory listing. Add the following to the .htaccess file at root level or to any directory where we want to avoid directory listing:
   ```json
      <Directory /var/www/mysite>
         Options -Indexes
      </Directory>
   ```

### Mitigation: Preventing Brute Force Attacks
1. Alarms:
   - Alert when there???s more than 15 failed attempts per minute to login into a user???s account. 
   - Alert when there???s more than 15 failed attempts per minute to login from the same IP address.
2. System Hardening:
   - Establish strong password policies, set password length to a minimum of 12 characters and set its expiration to 3 months with the past 4 passwords not allowed as new password.
   - Implement multi factor authentication.
   - After 3 failed login attempts, use CAPTCHA to prevent automated attacks.
   - Send an HTTP 200 success code (instead of error codes) and direct users to a page explaining the failed password attempt (to fool some automated tools).
   - Lock out user accounts that unsuccessfully try to login more than 15 times in one minute, let the lockout duration be 10 minutes. 
   - Hashed passwords should be salted.


### Mitigation: Detecting the WebDAV Connection
1. Alarms:
   - Alert when any non-whitelisted IP attempts to connect to WebDAV. 
2. System Hardening:
   - Limit access to WebDAV to only whitelisted IP addresses. Add the following to the .htaccess file:
     ```json
        <Limit GET POST PUT>
             Order deny,allow
             Deny from all
             Allow from <IP address here>
             Allow from <IP address here>
             Allow from <IP address here>
        </Limit>
     ```
   - Maintain the WebDAV application up to date (check for patches every week and install them if available).


### Mitigation: Identifying Reverse Shell Uploads
1. Alarms:
   - Alert when any HTTP PUT request is sent to the server for executable files. 
2. System Hardening:
   - Install antivirus software which will screen all incoming files and send to quarantine any executables or known malware.
   - Disable server-side handlers for the WebDAV directory to prevent the execution of executable files. In the directory???s .htaccess file add:
     ```json
        <Directory /path/to/restrict>
            AddHandler default-handler
        </Directory>
     ```


