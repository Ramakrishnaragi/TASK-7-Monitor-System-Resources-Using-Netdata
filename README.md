# TASK-7-Monitor-System-Resources-Using-Netdata

# Objective:
- Install and run Netdata using Docker to monitor system metrics like CPU, memory, disk, and Docker containers, and configure email alerts.
# Step-by-Step: Install and Run Netdata with Docker:
- Launch the ec2 instance (ubuntu)
# Install Docker
```sh
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```
# Run Netdata Container:
```sh
docker run -d --name=netdata \
  -p 19999:19999 \
  --cap-add=SYS_PTRACE \
  --security-opt apparmor=unconfined \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  netdata/netdata
```
- docker ps
  
![image](https://github.com/user-attachments/assets/8fa7ba28-ca25-43db-b6d7-e6cabb46f96e)

# Access Dashboard: Open your browser and go to:
- http://public_ip:19999

![image](https://github.com/user-attachments/assets/843d8072-832f-407b-aedb-9157a319310c)


# Using Netdata Dashboard
- Monitor Metrics:
```sh
CPU usage
RAM & swap
Disk I/O
Network interfaces
Docker container stats (e.g., CPU, memory, network)
```
- Explore Panels:
```sh
  > Click charts to zoom in and filter
  > Use the left panel to navigate between system components
```
- Alerts:
```
  > See active alarms and thresholds under the “Alarms” tab
```
- View Logs: Inside container
```
docker exec -it netdata bash
cd /var/log/netdata
tail -f error.log
```

# SET UP msmtp TO SEND EMAILS FROM NETDATA (Docker Host)
-  sudo apt update
-  sudo apt install msmtp msmtp-mta -y
# Create a Config File for msmtp: 
- Create the file (on the host, not in the container): nano ~/.msmtprc
- Paste this config (replace the placeholders with your Gmail info):
```
# Gmail SMTP configuration
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account        gmail
host           smtp.gmail.com
port           587
from           yourgmail@gmail.com
user           yourgmail@gmail.com
password  your_app_password

account default : gmail
```
- Set File Permissions: chmod 600 ~/.msmtprc (This protects your email credentials.)
- Set Netdata to Use msmtp for Email:
```
   > Inside container:
      - docker exec -it netdata bash
     - nano /etc/netdata/health_alarm_notify.conf
```
  > set:
    ```sh
    SEND_EMAIL="YES"
    EMAIL_CMD="msmtp -C /etc/netdata/.msmtprc"
    DEFAULT_RECIPIENT_EMAIL="yourgmail@gmail.com"
   ```
# Test Email from Command Line
- Run this test:
   - echo "Test email from Netdata" | msmtp yourgmail@gmail.com
- Check your inbox. If it arrives, you’re good to go.

![image](https://github.com/user-attachments/assets/c11f097e-90ad-4d1f-a42f-0c6739033fda)

# Install stress
- sudo apt update
- sudo apt install stress -y
- Run stress command for 60seconds
    - stress --cpu 4 --timeout 60

![image](https://github.com/user-attachments/assets/13d1a763-e17e-46fd-abf6-83545205d3ef)

# Explore Logs
-  If running inside Docker:
    -  Enter the container:
          -  docker exec -it netdata bash
- Navigate to the log directory:
    - cd /var/log/netdata
    - ls -lh

![image](https://github.com/user-attachments/assets/589721da-a671-42c7-a01d-df4492614c90)


# FINALLY, ALARMS:
- Onces the threshold range will reached means it shows the alert like this

![image](https://github.com/user-attachments/assets/a3a918c0-48d5-4dfc-9df6-8750829f7ee3)


# THANKYOU
