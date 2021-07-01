
# RedvsBlue
Project where the role of a pentester and a SOC analyst are explored


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
   - After finding out that “ashton” is a valid username, and that there is a secret folder that needs his credentials, we used Hydra and the rockyou dictionary to brute force the password and gain access to that directory.
   
2. Achievements:
   - We were able to find the password in approximately two minutes:
      - Username: ashton
      - Password: leopoldo 
 
3. Screenshots:
   - Run Hydra in our Kali Machine: ```hydra -l ashton -P rockyou.txt -s 80 -f -vV 192.168.1.105 http-get  /company_folders/secret_folder```
![alt text](https://github.com/alesanv/RedvsBlue/blob/main/Images/RedTeam/4_1_ashton_hydra.PNG "Hydra results")
  
### Exploitation: Weak Passwords #2
1. Tools and Processes:
   - Using the credentials for “ashton”, we had access to the company’s secret_folder and found a file containing a hashed password for username ryan.
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


