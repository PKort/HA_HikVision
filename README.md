# 0. Home Assistant + HikVision + go2rtc

Integrating HikVision video doorbell within Home Assistant dashboard running on Android tablet using go2rtc for 2-way audio.

I created this repository to gather all information related to integration of  HikVision video doorbell into my HomeAssistant-driven "smart home". Hopefully someone will find this information useful.

# 1. Hardware

Regarding hardware used:
1) HikVision DS-KV8113WME1(C) video doorbell
2) Home Assistant installed in Supervised mode on PC running Debian Linux
3) Xiaomi Redmi Pad Pro tablet to be used as dashboard

# 2. Pre-requisites

I had following items configured in Home Asssistant
1) SSL certficate - required to use 2-way audio in web browser. I use certificate from Let's Encrypt, installed on machine running Home Assistant. HA is available via HTTPS by NGINX reverse proxy. I chose port 8124. Config file attached as homeassistant-webgui
2) MQTT
3) Browser-mod - I use browser-mod popup to display video from doorbell on dashboard: 
   https://github.com/thomasloven/hass-browser_mod
Additionally I used iVMS-4200 installed on my Windows machine as well as Hik-Connect app on my phone for configuration of the doorbell.

# 3. Configuration

## 3.1 Doorbell configuration

I configured HikVision doorbell already long time ago, I don't remember exact steps I took. I followed steps from YT tutorials.
I ended up with doorbell configured as below:
1) Accessible from WWW via admin+password
2) Connected to my local Wi-Fi (I use mesh system to ensure proper signal coverage in lab as well as in target location
3) Assigned static IP adress
4) Connected to Hik-Connect cloud and my account.

## 3.2 Hik-Connect app

Doorbell is connected to Hik-Connect app. Whenever call button is pressed, all devices with Hik-Connect installed (and logged in to my account) receive a call and can answer.

## 3.3 Home Assistant 

I have done following configuration on my Home Assistant instance:

### 3.3.1 HikVision Add-On

I installed HikVision AddOn from here:
https://github.com/pergolafabio/Hikvision-Addons/
I added following new repo to the AddOn store:
https://github.com/pergolafabio/Hikvision-Addons
After refresh I was able to select and install the Add-On.
Configuration is pretty simple:

- name: Furtka
  ip: $DOORBELL_STATIC_IP
  username: admin
  password:$DOORBELL_ADMIN_PASSWORD

After starting the addon look for the line like below:
[2025-02-01 10:47:29.322][INF] **Private connect $DOORBELL_STATIC_IP:8000 sock=169 this=0x8d3050c4 cmd=0x10100 port=50172**
2025-02-01 10:47:30.547 | DEBUG    | config:load_mqtt_config:129 - Loading MQTT configuration from supervisor
2025-02-01 10:47:30.548 | DEBUG    | config:mqtt_config_from_supervisor:36 - Requesting MQTT service configuration to supervisor
2025-02-01 10:47:30.556 | INFO     | sdk.utils:loadSDK:44 - Using OS: Linux with architecture: x86_64
loop[2] find 2 mac and 1 ip
[2025-02-01 10:47:30.613][DBG] CCoreGlobalCtrlBase::LoadDSo, HPR_LoadDSo Succ, Path[/lib/x86_64-linux-gnu/libz.so.1.2.11], hHandleRet[170114560]
[2025-02-01 10:47:30.617][DBG] CCoreGlobalCtrlBase::LoadDSo, HPR_LoadDSo Succ, Path[/app/lib-amd64/libcrypto.so], hHandleRet[173746768]
[2025-02-01 10:47:30.617][DBG] CCoreGlobalCtrlBase::LoadDSo, HPR_LoadDSo Succ, Path[/app/lib-amd64/libssl.so], hHandleRet[173768064]
2025-02-01 10:47:30.677 | INFO     | doorbell:authenticate:82 - **Connected to doorbell: Furtka**
2025-02-01 10:47:30.677 | INFO     | event:__init__:87 - Setting up event handler: Console stdout
2025-02-01 10:47:30.678 | INFO     | mqtt:__init__:114 - Setting up event handler: MQTT

### 3.3.2 MQTT

Once Add-On is running, you should see the doorbell device in MQTT. For me it is the following:

![obraz](https://github.com/user-attachments/assets/a363759d-4cd4-4c61-89f6-411a9241ac68)

The most important are:
Call State - usually "idle", changes to "calling" when Call button is pressed on the doorbell. 
Relay - allows to open the lock 
Answer call / Hangup call / Reject call - used to perform operations on the call

### 3.3.3 go2rtc

I decied to install go2rtc as Add-On + integration.
To install the Add-On, following repo needs to be added to the AddOn Store:
https://github.com/AlexxIT/hassio-addons
After refresh AddOn should be visible and can be selected and installed. During installation AddOn creates its own configuration file go2rtc.yaml in the HA directory.


