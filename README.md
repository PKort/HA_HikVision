# 0. Home Assitant - HikVision 
Integrating HikVision video doorbell within Home Assistant dashboard running on Android tablet

I created this repository to keep all information related to integration of  HikVision video doorbell into my HomeAssistant-driven "smart home".

# 1. Hardware
Regarding hardware used:
1) HikVision DS-KV8113WME1(C) video doorbell
2) Home Assistant installed in Supervised mode on PC running Debian Linux
3) Xiaomi Redmi Pad Pro tablet to be used as dashboard

# 2. Pre-requisites
Following items configured in Home Asssistant
1) SSL certficate - required to use 2-way audio in web browser. I use certificate from Let's Encrypt, installed on machine running Home Assistant. HA is available via HTTPS by NGINX reverse proxy. I chose port 8124. Below entry :
   
3) Browser-mod - I use browser-mod popup to display video from doorbell on dashboard:
   
   https://github.com/thomasloven/hass-browser_mod
4) 

Additionally I used iVMS-4200 installed on my Windows machine for initial configuration of the doorbell.

# 3. Conifguration

## 3.1 Doorbell configuration
