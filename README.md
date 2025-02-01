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

![obraz](https://github.com/user-attachments/assets/89d79a61-d650-47d4-bd67-affebb13581e)

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

- Call State - usually "idle", changes to "calling" when Call button is pressed on the doorbell. 
- Relay - allows to open the lock 
- Answer call / Hangup call / Reject call - used to perform operations on the call

### 3.3.3 Additional commands in HA machine

I added link to my SSL certificate so it can be available for HA. I logged to my HA machine, elevated to root and ran following commands:

    cd /usr/share/hassio/homeassistant
    ln -s ln -s /etc/letsencrypt/live/$MYDOMAINNAME/ ssl

### 3.3.3 go2rtc Add-On

I decied to install go2rtc as Add-On + integration.
To install the Add-On, following repo needs to be added to the AddOn Store:

   https://github.com/AlexxIT/hassio-addons

After refresh AddOn should be visible and can be selected and installed. During installation AddOn creates its own configuration file go2rtc.yaml in the HA directory. I have included file <go2rtc.yaml> with my configuration.

I use ufw as local firewall on HA machine. I had to open some ports for go2rtc:

    ufw allow 1985 comment "go2rtc API Server SSL"
    ufw allow 8554 comment "go2rtc RTSP Server"
    ufw allow 8555 comment "go2rtc WebRTC Server"
    systemctl restart ufw

After running the AddOn, its UI can be accessed by web browser by navigating to https://$MYDOMAINNAME:11985 and logging with username admin, password $GO2RTC_ADMIN_PASSWORD
In my case, doorbell defined in the YAML file is visible as below:

![obraz](https://github.com/user-attachments/assets/c5bc0eb2-bed3-40e8-a0b7-b1bf06ab1f37)

Some tutorials can be found in YouTube which cover this part of configuration. In some of these tutorials, "Commands" section contain the "2-way aud" button which allows you to quickly get the link for the stream with 2-way audio enabled which speeds up the setup. This was not the case for me.

Instead, I navigated to "links" and found the following:

![obraz](https://github.com/user-attachments/assets/c0fa41ae-d910-48e8-90d1-4e31d1a6ccf6)

When I select the "video+audio+microphone" option, right-click on webrtc.html returns direct link to camera stream with 2-way audio. You need to access it via HTTPS, otherwise browsers will block use of local microphone. But if you open via HTTPS, browser will ask permission to use microphone and you can use mic and speakers of your device to communicate with doorbell.


