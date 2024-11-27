# Lab #1,21110011, Tran Dai Duong, INSE331280E_03FIE
# Task 1: Encrypt and Decrypt Text file

# Task 1: Firewall configuration 
**Question 1**: 
Setup a set of vms/containers in a network configuration of 2 subnets (1,2) with a router forwarding traffic between them. Relevant services are also required:
- The router is initially can not route traffic between subnets
- PC0 on subnet 1 serves as a web server on subnet 1
- PC1,PC2 on subnet 2 acts as client workstations on subnet 2 
**Answer 1**:

Create new subnets : 
```sh
docker network create --subnet=192.168.1.0/24 subnet1
docker network create --subnet=192.168.2.0/24 subnet2
```
![image](https://github.com/user-attachments/assets/60fceb9b-5737-4373-803c-d84fd32bea5c)
Create Router Container
```sh
docker run -d --name router --net bridge --privileged --entrypoint "/bin/bash" ubuntu sleep infinity
```
Add interfaces for the router to each subnet" 
```sh
docker network connect subnet1 router
docker network connect subnet2 router
```
![image](https://github.com/user-attachments/assets/d7c6970a-37e7-44f8-9d40-3503fb3b4eaa)

Initial Router Settings:: 
```sh
docker exec -it router bash
echo 0 > /proc/sys/net/ipv4/ip_forward  # Disable IP forwarding initially
exit
```
![image](https://github.com/user-attachments/assets/3b8f873b-0219-4c4a-a736-10de17a73b2d)

Create Containers for PC0, PC1 and PC2:
PC0 (Web Server): This container will be on Subnet 1.
PC1, PC2 (Clients): These containers will be on Subnet 2.
```sh
docker run -d --name pc0 --net subnet1 --ip 192.168.1.2 nginx
docker run -d --name pc1 --net subnet2 --ip 192.168.2.2 ubuntu sleep infinity
docker run -d --name pc2 --net subnet2 --ip 192.168.2.3 ubuntu sleep infinity
```
![image](https://github.com/user-attachments/assets/30f7998e-8f8b-42d7-888f-bace6be9b191)

**Question 2**:
- Enable packet forwarding on the router.
- Deface the webserver's home page with ssh connection on PC1
  
**Answer 2**:
Enable IP Forwarding on Router:
```sh
docker exec -it router bash
echo 1 > /proc/sys/net/ipv4/ip_forward  
exit
```
![image](https://github.com/user-attachments/assets/ae598fca-4004-4d7e-b771-0edb0ab6523d)
Deface Web Server Home Page
- First, SSH into PC1 (running on Subnet 2).
- Modify the homepage of the web server (PC0).
```sh
docker exec -it pc1 bash
docker exec -it pc0 bash
echo "Defaced Web Page" > /usr/share/nginx/html/index.html
exit
```
![image](https://github.com/user-attachments/assets/823637b2-e79e-4614-aceb-e783e9b626b5)

**Question 3**:
  Config the router to block ssh to web server from PC1, leaving ssh/web access normally for all other hosts from subnet 1.   

**Answer 3**:
Block SSH from PC1: Use iptables on the router to block SSH from PC1 to PC0 :
```sh
docker exec -it router bash
iptables -A FORWARD -s 192.168.2.2 -d 192.168.1.2 -p tcp --dport 22 -j REJECT
exit
```
![image](https://github.com/user-attachments/assets/0b7eb20e-4822-43d9-8948-29ba54c2ce9e)
Allow SSH/Web Access from Subnet 1: Ensure the router allows SSH and web traffic from other hosts on Subnet 1 :
```sh
docker exec -it router bash
iptables -A FORWARD -s 192.168.1.0/24 -d 192.168.1.2 -p tcp --dport 22 -j ACCEPT
iptables -A FORWARD -s 192.168.1.0/24 -d 192.168.1.2 -p tcp --dport 80 -j ACCEPT
exit
```
![image](https://github.com/user-attachments/assets/1b6d9842-fbb1-4bb0-8016-74cb4cb8e044)
![image](https://github.com/user-attachments/assets/7625fe1c-cf58-4b9e-b133-c63d6d256fe5)


**Question 4**:
- PC1 now servers as a UDP server, make sure that it can reply UDP ping from other hosts on both subnets.
- Config personal firewall on PC1 to block UDP accesses from PC2 while leaving UDP access from the server intact.
  
**Answer 4**:
Make PC1 a UDP Server: Start a simple UDP server on PC1:
```sh
docker exec -it pc1 bash
nc -lu 12345
```
![image](https://github.com/user-attachments/assets/2ca3a134-b254-414c-97e5-c16ce2fd620f)
Block UDP Access from PC2 to PC1: Use ufw or iptables on PC1 to block UDP access from PC2:
```sh
docker exec -it pc1 bash
ufw deny from 192.168.2.3 to any port 12345 proto udp
ufw allow from 192.168.2.0/24 to any port 12345 proto udp  # Allow other UDP traffic
exit
```
# Task 2: Encrypting large message 
Use PC0 and PC2 for this lab 
Create a text file at least 56 bytes on PC2 this file will be sent encrypted to PC0

![image](https://github.com/user-attachments/assets/eb179d0f-086d-4c59-a84e-327aa742dbc5)
Create a file on PC2: 
```sh
echo "This is a test message that is over 56 bytes long to test encryption!" > /tmp/testfile.txt
```
![image](https://github.com/user-attachments/assets/4580e5d0-2a3e-4f1a-91d9-3c97a79fcad2)

**Question 1**:
Encrypt the file with aes-cipher in CTR and OFB modes. How do you evaluate both cipher in terms of error propagation and adjacent plaintext blocks are concerned. 

**Answer 1**:
- Demonstrate your ability to send file to PC0 to with message authentication measure.
- Verify the received file for each cipher modes
  
  *AES CTR Mode Encryption*
    * On PC2, use AES encryption in CTR mode to encrypt the file.
      ```sh
      docker exec -it pc2 bash
      openssl enc -aes-256-ctr -in testfile.txt -out /tmp/encrypted_ctr.bin -pass pass:mysecretkey
      exit
      ```
  *AES OFB Mode Encryption*
     * On PC2, encrypt the file in AES-OFB mode
       ```sh docker exec -it pc2 bash
          openssl enc -aes-256-ofb -in /tmp/testfile.txt -out /tmp/encrypted_ofb.bin -pass pass:mysecretkey
          exit
       ```
       ![image](https://github.com/user-attachments/assets/ad968bb0-4276-4326-ae17-1c0e16a923f4)
       ![image](https://github.com/user-attachments/assets/9d1881a3-4825-48d3-8262-2238cda1596c)
  *Message Authentication:*
    ```sh
    docker exec -it pc2 bash
    openssl dgst -sha256 -mac HMAC -macopt key:mysecretkey /tmp/testfile.txt
    exit
    ```
    ![image](https://github.com/user-attachments/assets/99778b7c-61ff-4d7c-ab27-de4200f5dd1e)

    Send the encrypted file along with the HMAC to PC0: Use netcat
    ```sh
    cat /tmp/encrypted_ctr.bin | nc 192.168.1.2 12345
    ```
    Receive and Verify the File on PC0:
    ```sh
    nc -l -p 12345 > /tmp/encrypted_ctr_received.bin
    openssl dgst -sha256 -mac HMAC -macopt key:mysecretkey /tmp/encrypted_ctr_received.bin
    ```

**Question 2**:
- Assume the 6th bit in the ciphered file is corrupted.
- Verify the received files for each cipher mode on PC0

**Answer 2**:
Corrupt the Cipher Text: On PC0, manually corrupt the 6th bit of the encrypted file.
```sh
docker exec -it pc0 bash
xxd -p /tmp/encrypted_ctr_received.bin | sed 's/\(..\)\(..\)\(..\)/\1\3\2/' | xxd -r -p > /tmp/encrypted_ctr_corrupted.bin
exit
```
Verify the Corrupted File 
```sh
openssl enc -d -aes-256-ctr -in /tmp/encrypted_ctr_corrupted.bin -out /tmp/decrypted_ctr_corrupted.txt -pass pass:mysecretkey
```
**Question 3**:
- Decrypt corrupted files on PC0.
- Comment on both ciphers in terms of error propagation and adjacent plaintext blocks criteria.
  
**Answer 3**:
Decrypt the Corrupted Files: 
```sh
# Decrypt corrupted file in AES-CTR mode
docker exec -it pc0 bash
openssl enc -d -aes-256-ctr -in /tmp/encrypted_ctr_corrupted.bin -out /tmp/decrypted_ctr_corrupted.txt -pass pass:mysecretkey
exit

# Decrypt corrupted file in AES-OFB mode
docker exec -it pc0 bash
openssl enc -d -aes-256-ofb -in /tmp/encrypted_ofb_corrupted.bin -out /tmp/decrypted_ofb_corrupted.txt -pass pass:mysecretkey
exit

```
Final Comments:
- CTR Mode: This mode offers better error containment because any corruption that occurs in one block will only affect that specific block. The rest of the data remains unaffected, as errors do not propagate to adjacent blocks.
- OFB Mode: In contrast, errors in OFB mode can have a wider impact. A single corrupted bit in the ciphertext will affect the corresponding decrypted bit, and this error will spread across all subsequent blocks, affecting the entire decryption sequence until the error is corrected.



