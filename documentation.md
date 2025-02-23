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

  - I opened the terminal and typed the command, "wpscan --url https://bricks.thm/".
      - The scan aborted, "url seems to be down, SSL peer certificate or SSH remote key was not OK".
      - Retried scan while disabling the tls certificate check, "wpscan --url https://bricks.thm/ --disable-tls-checks".
        
  - The scan ran successfully, the results showed the style "bricks" for WordPress, and the version was 1.9.5.

  - I used Google to search for "bricks 1.9.5" and the results showed that this was a vulnerable version of bricks, and there was an exploit available for this vulnerability (CVE-2024-25600) on GitHub.

    ### Step 2: Exploitation

  - I copied the raw exploit code for the bricks version 1.9.5 vulnerability from GitHub and pasted it into a nano .txt file titled, "shell.py".

  - Opened a terminal and ran the command, ```python3 shell.py -u https://bricks.thm```
      - There was an error running the script, a few modules needed to be installed first.
          - Installed rich module, ```pip3 install rich```
              - Retried the python script, another error occured.
                  - Installed prompt-toolkit module, ```pip3 install prompt-toolkit```
                      - Retried the python script, yet another error occured.
                        - Installed alive-progress module, ```pip3 install alive-progress```
                            - Ran python script, it finally worked and the vulnerability was confirmed to be present.
  
  ### Step 3: Exploring Initial Access

  - I ran the command "whoami" to discover who I was logged into.
