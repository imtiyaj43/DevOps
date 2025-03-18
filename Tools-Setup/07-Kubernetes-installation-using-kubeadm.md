## Step 0 : System Requirements :

### 1. OS : Ubuntu 24.04 LTS Server
### 2. RAM : 2GB or more
### 3. Disk : 20GB+
### 4. Swap Memory : Disable

## Step 1 : Disable Swap Temporarily
```
sudo swapoff -a
```

## Step 2 : Disable Swap Permanently
```
sudo vi /etc/fstab
```
### #Comment the line : UUID=<some-uuid> none swap sw 0 
(If Not show above line, No problem)
