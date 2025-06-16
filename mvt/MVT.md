# Mobile Verification Toolkit
Consensual forensics analysis of mobile devices to detect traces of compromise.

## mvt-android
For this, we test mvt-android in two scenario. 
1. Using google's high-fidelity emulators such as Cuttlefish
2. Using real phone through USB adb

### Setting developer/tester environment for cuttlefish

```bash
sudo apt update && \
sudo apt install -y git devscripts equivs config-package-dev debhelper-compat golang curl && \
cd "$HOME" && \
if [ ! -d android-cuttlefish ]; then git clone https://github.com/google/android-cuttlefish; else echo "android-cuttlefish exists, skipping clone"; fi && \
cd android-cuttlefish && \
tools/buildutils/build_packages.sh && \
sudo dpkg -i ./cuttlefish-base_*_*64.deb || (echo "Fixing dependencies and retrying cuttlefish-base install..." && sudo apt-get install -f -y && sudo dpkg -i ./cuttlefish-base_*_*64.deb) && \
sudo dpkg -i ./cuttlefish-user_*_*64.deb || (echo "Fixing dependencies and retrying cuttlefish-user install..." && sudo apt-get install -f -y && sudo dpkg -i ./cuttlefish-user_*_*64.deb) && \
sudo usermod -aG kvm,cvdnetwork,render "$USER" && \
echo "All done! Please reboot your machine for group changes to take effect."

```

Above installed the Cuttlefish daemon, libraries, and utilities to your system (likely in /usr/bin, /usr/lib, etc.) 
Set up udev rules and kernel modules needed for running the emulator. 
You can run cvd, which is cuttlefish virtual device manager tool. 

Official Google build many types of Cuttlefish image here: https://ci.android.com

GSI means Generic System Image (this is offical unmodified AOSP, no MIUI, no OneUI etc.)

```angular2html
Naming syntax: aosp_cf_<arch>[_only]_phone[-variant]
    Where:
    cf → Cuttlefish (emulator build)
    <arch> → x86_64, arm64, etc.
    phone → form factor (can also be tablet, tv, etc.)
    only → indicates a reduced or minimal configuration
    -variant → e.g., user, userdebug, etc.
```

Note: if your host system is x86 then install x86 emulator. Different architecture type demands complete emulation, 
which is very slow. 

```bash
upgautam@amd:/opt$ uname -m
x86_64 # I will download phone only x86-64 cuttlefish emulator
```
I downloaded aosp_cf_x86_64_phone_userdebug 12. For this I need to download two artifacts (this screen comes 
after you click download at the first screen). In ci.android.com, search "aosp-android12-gsi" 
and click to download arrow under android_aosp_x86_64_only_phone. There download two artifacts. 
1. https://ci.android.com/builds/submitted/13439841/aosp_cf_x86_64_only_phone-userdebug/latest/aosp_cf_x86_64_only_phone-img-13439841.zip
2. https://ci.android.com/builds/submitted/13439841/aosp_cf_x86_64_only_phone-userdebug/latest/cvd-host_package.tar.gz

Now, put extract then so that their contents are combined (i.e., mixed or merged) in ~/android-cuttlefish/cf/

Now, add your user into group virtaccess
```bash
sudo groupadd virtaccess && \
sudo usermod -aG virtaccess $USER
```

Now, launch cvd emulator as,
```bash
HOME=$PWD ./bin/launch_cvd --daemon
```

**Note**: In cuttlefish emulator, no need to enable developer options (it is already enabled by default). 
You can also enable developer options and enabled USB debugging in your real phone. 

**Note**: Always use Google Chrome or Chromium to access webrtc acces (e.g., https://localhost:8443/)

### Mobile Verification Tool (MVT)
Mobile Verification Toolkit (MVT) is a collection of utilities designed to facilitate the consensual forensic acquisition 
of iOS and Android devices for the purpose of identifying any signs of compromise. MVT's capabilities are continuously 
evolving, but some of its key features include:
1. Decrypt encrypted iOS backups. 
2. Process and parse records from numerous iOS system and apps databases, logs and system analytics. 
3. Extract installed applications from Android devices. 
4. Extract diagnostic information from Android devices through the adb protocol. 
5. Compare extracted records to a provided list of malicious indicators in STIX2 format. 
6. Generate JSON logs of extracted records, and separate JSON logs of all detected malicious traces. 
7. Generate a unified chronological timeline of extracted records, along with a timeline all detected malicious traces.

Official links: 
1. https://github.com/mvt-project/mvt
2. https://docs.mvt.re

#### We install from PyPI. 
```bash
sudo apt update && \
sudo apt install -y python3 python3-venv python3-pip sqlite3 libusb-1.0-0 pipx && \
pipx ensurepath && \
pipx install mvt
```

Now, you can run mvt-android as, 
```angular2html
upgautam@amd:~/CLionProjects/ITSL_Summer_Research_Uddhav$ mvt-android
Usage: mvt-android [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  check-adb        Check an Android device over ADB
  check-androidqf  Check data collected with AndroidQF
  check-backup     Check an Android Backup
  check-bugreport  Check an Android Bug Report
  check-iocs       Compare stored JSON results to provided indicators
  download-apks    Download all or only non-system installed APKs
  download-iocs    Download public STIX2 indicators
  version          Show the currently installed version of MVT
```

#### Testing
The first thing you do is downloading iocs. IOCs are .stix2 files which are indicator of compromise. 
They are downloaded here: /home/upgautam/.local/share/mvt/indicators/

```angular2html
upgautam@amd:~/android-cuttlefish/cf$ adb devices
List of devices attached
0.0.0.0:6520    device

upgautam@amd:~/android-cuttlefish/cf$ mvt-android download-iocs


MVT - Mobile Verification Toolkit
https://mvt.re
Version: 2.6.1
Indicators updates checked recently, next automatic check in 11 hours


13:30:27 INFO     [mvt.common.updates] Downloaded indicators "NSO Group Pegasus Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2021-07
-18_nso_pegasus.stix2
INFO     [mvt.common.updates] Downloaded indicators "Predator Spyware Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_intellexa
_predator_predator.stix2
INFO     [mvt.common.updates] Downloaded indicators "RCS Lab Spyware Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2022-06-2
3_rcs_lab_rcs.stix2
INFO     [mvt.common.updates] Downloaded indicators "Stalkerware Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AssoEchap_stalkerware-indicators_master_g
enerated_stalkerware.stix2
INFO     [mvt.common.updates] Downloaded indicators "Surveillance campaign linked to mercenary spyware company" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2023-03
-29_android_campaign_malware.stix2
INFO     [mvt.common.updates] Downloaded indicators "Quadream KingSpawn Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-04-1
1_quadream_kingspawn.stix2
INFO     [mvt.common.updates] Downloaded indicators "Operation Triangulation Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-06_0
1_operation_triangulation_operation_triangulation.stix2
INFO     [mvt.common.updates] Downloaded indicators "WyrmSpy and DragonEgg Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-07-2
5_wyrmspy_dragonegg_wyrmspy_dragonegg.stix2
13:30:28 INFO     [mvt.common.updates] Downloaded indicators "Wintego Helios Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2024-05
-02_wintego_helios_wintego_helios.stix2
INFO     [mvt.common.updates] Downloaded indicators "NoviSpy (Serbia) Indicators of Compromise" to
/home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2024-12
-16_serbia_novispy_novispy.stix2        
```
From the official docs _"Unfortunately Android devices provide much less observability than their iOS cousins. Android stores very little diagnostic information useful to triage potential compromises, and because of this mvt-android capabilities are limited as well."_

Before testing, I side loaded download firefox apk. 
```angular2html
upgautam@amd:~/android-cuttlefish/cf$ adb -s 0.0.0.0:6520 push ./fenix-110.0.1-x86_64.apk /data/local/tmp/
./fenix-110.0.1-x86_64.apk: 1 file pushed, 0 skipped. 630.7 MB/s (89851047 bytes in 0.136s)
upgautam@amd:~/android-cuttlefish/cf$ adb -s 0.0.0.0:6520 shell pm install -r /data/local/tmp/fenix-110.0.1-x86_64.apk
Success
```

Now, we can run 
```bash
upgautam@amd:~/android-cuttlefish/cf$ mvt-android check-adb --serial 0.0.0.0:6520 --iocs /home/upgautam/.local/share/mvt/indicators
```

Output, 
```angular2html
upgautam@amd:~/android-cuttlefish/cf$ mvt-android check-adb --serial 0.0.0.0:6520 --iocs /home/upgautam/.local/share/mvt/indicators


        MVT - Mobile Verification Toolkit
                https://mvt.re
                Version: 2.6.1
                Indicators updates checked recently, next automatic check in 12 hours


13:42:11 WARNING  [mvt.android.cmd_check_adb] No indicators file exists at path /home/upgautam/.local/share/mvt/indicators                                                                   
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2021-07-18_nso_pegasus.stix2                                        
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_intellexa_predator_predator.stix2                                     
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2022-06-23_rcs_lab_rcs.stix2                                          
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AssoEchap_stalkerware-indicators_master_generated_stalkerware.stix2                                   
13:42:12 INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2023-03-29_android_campaign_malware.stix2                           
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-04-11_quadream_kingspawn.stix2                                   
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-06_01_operation_triangulation_operation_triangulation.stix2      
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_mvt-project_mvt-indicators_main_2023-07-25_wyrmspy_dragonegg_wyrmspy_dragonegg.stix2                  
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2024-05-02_wintego_helios_wintego_helios.stix2                      
         INFO     [mvt.android.cmd_check_adb] Parsing STIX2 indicators file at path                                                                                                          
                  /home/upgautam/.local/share/mvt/indicators/raw.githubusercontent.com_AmnestyTech_investigations_master_2024-12-16_serbia_novispy_novispy.stix2                             
         INFO     [mvt.android.cmd_check_adb] Loaded a total of 10722 unique indicators                                                                                                      
         INFO     [mvt] Checking Android device over debug bridge                                                                                                                            
         INFO     [mvt.android.modules.adb.chrome_history] Running module ChromeHistory...                                                                                                   
13:42:13 ERROR    [mvt.android.modules.adb.chrome_history] Unable to download file /sdcard/tmp_U7TESFHrL8: Command failed: open failed: No such file or directory                            
         INFO     [mvt.android.modules.adb.chrome_history] The ChromeHistory module produced no detections!                                                                                  
         INFO     [mvt.android.modules.adb.sms] Running module SMS...                                                                                                                        
         ERROR    [mvt.android.modules.adb.sms] Error in running extraction from module SMS: Unable to download file /sdcard/tmp_SPkg4OJBtI: Command failed: open failed: No such file or    
                  directory                                                                                                                                                                  
                  Traceback (most recent call last):                                                                                                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 219, in _adb_download                               
                      self.device.pull(remote_path, local_path, progress_callback)                                                                                                           
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 944, in pull                                                
                      self._pull(device_path, stream, progress_callback, adb_info, filesync_info)                                                                                            
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 969, in _pull                                               
                      for cmd_id, _, data in self._filesync_read_until([constants.DATA], [constants.DONE], adb_info, filesync_info):                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 1433, in _filesync_read_until                               
                      cmd_id, header, data = self._filesync_read(expected_ids + finish_ids, adb_info, filesync_info)                                                                         
                                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^                                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 1372, in _filesync_read                                     
                      raise exceptions.AdbCommandFailureException('Command failed: {}'.format(reason))                                                                                       
                  adb_shell.exceptions.AdbCommandFailureException: Command failed: open failed: No such file or directory                                                                    
                                                                                                                                                                                             
                  During handling of the above exception, another exception occurred:                                                                                                        
                                                                                                                                                                                             
                  Traceback (most recent call last):                                                                                                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 219, in _adb_download                               
                      self.device.pull(remote_path, local_path, progress_callback)                                                                                                           
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 944, in pull                                                
                      self._pull(device_path, stream, progress_callback, adb_info, filesync_info)                                                                                            
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 969, in _pull                                               
                      for cmd_id, _, data in self._filesync_read_until([constants.DATA], [constants.DONE], adb_info, filesync_info):                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 1433, in _filesync_read_until                               
                      cmd_id, header, data = self._filesync_read(expected_ids + finish_ids, adb_info, filesync_info)                                                                         
                                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^                                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/adb_shell/adb_device.py", line 1372, in _filesync_read                                     
                      raise exceptions.AdbCommandFailureException('Command failed: {}'.format(reason))                                                                                       
                  adb_shell.exceptions.AdbCommandFailureException: Command failed: open failed: No such file or directory                                                                    
                                                                                                                                                                                             
                  The above exception was the direct cause of the following exception:                                                                                                       
                                                                                                                                                                                             
                  Traceback (most recent call last):                                                                                                                                         
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/common/module.py", line 171, in run_module                                             
                      exec_or_profile("module.run()", globals(), locals())                                                                                                                   
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/common/utils.py", line 263, in exec_or_profile                                         
                      exec(module, globals, locals)                                                                                                                                          
                    File "<string>", line 1, in <module>                                                                                                                                     
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/sms.py", line 159, in run                                          
                      self._adb_process_file(                                                                                                                                                
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 298, in _adb_process_file                           
                      self._adb_download(new_remote_path, local_path)                                                                                                                        
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 222, in _adb_download                               
                      self._adb_download_root(remote_path, local_path, progress_callback)                                                                                                    
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 261, in _adb_download_root                          
                      self._adb_download(                                                                                                                                                    
                    File "/home/upgautam/.local/share/pipx/venvs/mvt/lib/python3.12/site-packages/mvt/android/modules/adb/base.py", line 224, in _adb_download                               
                      raise Exception(                                                                                                                                                       
                  Exception: Unable to download file /sdcard/tmp_SPkg4OJBtI: Command failed: open failed: No such file or directory                                                          
         INFO     [mvt.android.modules.adb.whatsapp] Running module Whatsapp...                                                                                                              
         ERROR    [mvt.android.modules.adb.whatsapp] Unable to download file /sdcard/tmp_jpmyyyJXho: Command failed: open failed: No such file or directory                                  
         INFO     [mvt.android.modules.adb.whatsapp] The Whatsapp module produced no detections!                                                                                             
         INFO     [mvt.android.modules.adb.processes] Running module Processes...                                                                                                            
13:42:14 INFO     [mvt.android.modules.adb.processes] Extracted records on a total of 306 processes                                                                                          
         INFO     [mvt.android.modules.adb.processes] The Processes module produced no detections!                                                                                           
         INFO     [mvt.android.modules.adb.getprop] Running module Getprop...                                                                                                                
         INFO     [mvt.android.modules.adb.getprop] Extracted 483 Android system properties                                                                                                  
         INFO     [mvt.android.modules.adb.getprop] gsm.sim.operator.alpha: TelAlaska Cellular                                                                                               
         INFO     [mvt.android.modules.adb.getprop] gsm.sim.operator.iso-country: us                                                                                                         
         INFO     [mvt.android.modules.adb.getprop] persist.sys.timezone: America/New_York                                                                                                   
         INFO     [mvt.android.modules.adb.getprop] ro.boot.serialno: CUTTLEFISHCVD011                                                                                                       
         INFO     [mvt.android.modules.adb.getprop] ro.build.version.sdk: 31                                                                                                                 
         INFO     [mvt.android.modules.adb.getprop] ro.build.version.security_patch: 2023-01-01                                                                                              
         WARNING  [mvt.android.modules.adb.getprop] This phone has not received security updates for more than six months (last update: 2023-01-01)                                          
         INFO     [mvt.android.modules.adb.getprop] ro.product.cpu.abi: x86_64                                                                                                               
         INFO     [mvt.android.modules.adb.getprop] ro.product.locale: en-US                                                                                                                 
         INFO     [mvt.android.modules.adb.getprop] ro.product.vendor.manufacturer: Google                                                                                                   
         INFO     [mvt.android.modules.adb.getprop] ro.product.vendor.model: Cuttlefish x86_64 phone 64-bit only                                                                             
         INFO     [mvt.android.modules.adb.getprop] ro.product.vendor.name: aosp_cf_x86_64_only_phone                                                                                        
         INFO     [mvt.android.modules.adb.getprop] The Getprop module produced no detections!                                                                                               
         INFO     [mvt.android.modules.adb.settings] Running module Settings...                                                                                                              
         WARNING  [mvt.android.modules.adb.settings] Found suspicious "secure" setting "install_non_market_apps = 1" (enabled installation of non Google Play apps)                          
         INFO     [mvt.android.modules.adb.settings] The Settings module produced no detections!                                                                                             
         INFO     [mvt.android.modules.adb.selinux_status] Running module SELinuxStatus...                                                                                                   
         INFO     [mvt.android.modules.adb.selinux_status] SELinux is being regularly enforced                                                                                               
         INFO     [mvt.android.modules.adb.selinux_status] The SELinuxStatus module does not support checking for indicators                                                                 
         INFO     [mvt.android.modules.adb.dumpsys_battery_history] Running module DumpsysBatteryHistory...                                                                                  
         INFO     [mvt.android.modules.adb.dumpsys_battery_history] Extracted 57 records from battery history                                                                                
         INFO     [mvt.android.modules.adb.dumpsys_battery_history] The DumpsysBatteryHistory module produced no detections!                                                                 
         INFO     [mvt.android.modules.adb.dumpsys_battery_daily] Running module DumpsysBatteryDaily...                                                                                      
         INFO     [mvt.android.modules.adb.dumpsys_battery_daily] Extracted 0 records from battery daily stats                                                                               
         INFO     [mvt.android.modules.adb.dumpsys_battery_daily] The DumpsysBatteryDaily module produced no detections!                                                                     
         INFO     [mvt.android.modules.adb.dumpsys_receivers] Running module DumpsysReceivers...                                                                                             
13:42:15 INFO     [mvt.android.modules.adb.dumpsys_receivers] Extracted receivers for 134 intents                                                                                            
         INFO     [mvt.android.modules.adb.dumpsys_receivers] Found a receiver to intercept incoming SMS messages: "com.android.messaging/.receiver.AbortSmsReceiver"                        
         INFO     [mvt.android.modules.adb.dumpsys_receivers] Found a receiver to intercept incoming SMS messages: "com.android.messaging/.receiver.SmsReceiver"                             
         INFO     [mvt.android.modules.adb.dumpsys_receivers] Found a receiver monitoring outgoing calls: "com.android.dialer/.interactions.UndemoteOutgoingCallReceiver"                    
         INFO     [mvt.android.modules.adb.dumpsys_receivers] The DumpsysReceivers module produced no detections!                                                                            
         INFO     [mvt.android.modules.adb.dumpsys_activities] Running module DumpsysActivities...                                                                                           
13:42:17 INFO     [mvt.android.modules.adb.dumpsys_activities] Extracted 510 package activities                                                                                              
         INFO     [mvt.android.modules.adb.dumpsys_activities] The DumpsysActivities module produced no detections!                                                                          
         INFO     [mvt.android.modules.adb.dumpsys_accessibility] Running module DumpsysAccessibility...                                                                                     
         INFO     [mvt.android.modules.adb.dumpsys_accessibility] Identified a total of 0 accessibility services                                                                             
         INFO     [mvt.android.modules.adb.dumpsys_accessibility] The DumpsysAccessibility module produced no detections!                                                                    
         INFO     [mvt.android.modules.adb.dumpsys_dbinfo] Running module DumpsysDBInfo...                                                                                                   
         INFO     [mvt.android.modules.adb.dumpsys_dbinfo] Extracted a total of 560 records from database information                                                                        
         INFO     [mvt.android.modules.adb.dumpsys_dbinfo] The DumpsysDBInfo module produced no detections!                                                                                  
         INFO     [mvt.android.modules.adb.dumpsys_adbstate] Running module DumpsysADBState...                                                                                               
         INFO     [mvt.android.modules.adb.dumpsys_adbstate] Identified a total of 0 trusted ADB keys                                                                                        
         INFO     [mvt.android.modules.adb.dumpsys_adbstate] The DumpsysADBState module produced no detections!                                                                              
         INFO     [mvt.android.modules.adb.dumpsys_full] Running module DumpsysFull...                                                                                                       
13:42:23 INFO     [mvt.android.modules.adb.dumpsys_full] The DumpsysFull module does not support checking for indicators                                                                     
         INFO     [mvt.android.modules.adb.dumpsys_appops] Running module DumpsysAppOps...                                                                                                   
         INFO     [mvt.android.modules.adb.dumpsys_appops] Extracted a total of 27 records from app-ops manager                                                                              
         INFO     [mvt.android.modules.adb.dumpsys_appops] The DumpsysAppOps module produced no detections!                                                                                  
         INFO     [mvt.android.modules.adb.packages] Running module Packages...                                                                                                              
13:42:57 INFO     [mvt.android.modules.adb.packages] Found non-system package with name "org.mozilla.firefox" installed by "None" on 2025-06-16 13:40:56                                     
Looking up 1 files... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   0% -:--:--
         INFO     [mvt.android.modules.adb.packages] Extracted at total of 116 installed package names                                                                                       
13:42:58 INFO     [mvt.android.modules.adb.packages] The Packages module produced no detections!                                                                                             
         INFO     [mvt.android.modules.adb.logcat] Running module Logcat...                                                                                                                  
13:43:04 INFO     [mvt.android.modules.adb.logcat] The Logcat module does not support checking for indicators                                                                                
         INFO     [mvt.android.modules.adb.root_binaries] Running module RootBinaries...                                                                                                     
13:43:05 WARNING  [mvt.android.modules.adb.root_binaries] Found root binary "su"                                                                                                             
         INFO     [mvt.android.modules.adb.files] Running module Files...                                                                                                                    
         INFO     [mvt.android.modules.adb.files] Found file in tmp folder at path /data/local/tmp/fenix-110.0.1-x86_64.apk                                                                  
         INFO     [mvt.android.modules.adb.files] Found 9 files in primary Android tmp and media folders                                                                                     
         INFO     [mvt.android.modules.adb.files] Processing full file listing. This may take a while...                                                                                     
13:43:19 INFO     [mvt.android.modules.adb.files] Found 106709 total files                                                                                                                   
         WARNING  [mvt.android.modules.adb.files] Found an SUID file in a non-standard directory "/system/xbin/su".                                                                          
13:43:21 INFO     [mvt.android.modules.adb.files] The Files module produced no detections!                                                                                                   
         INFO     [mvt.android.cmd_check_adb] Please disable Developer Options and ADB (Android Debug Bridge) on the device once finished with the acquisition. ADB is a powerful tool which 
                  can allow unauthorized access to the device.                                                                                                                               
         WARNING   NOTE: Detected indicators of compromise. Only expert review can confirm if the detected indicators are signs of an attack.                                                
                                                                                                                                                                                             
                  Please seek reputable expert help if you have serious concerns about a possible spyware attack. Such support is available to human rights defenders and civil society      
                  through Amnesty International's Security Lab at https://securitylab.amnesty.org/get-help/?c=mvt                                                                            
         WARNING  [mvt] The analysis of the Android device produced 1 detections!    
```



One main problem is the device need to be rooted for most of the check-adb functionalities to be working. 
You could even save above log as ` mvt-android check-adb --serial 0.0.0.0:6520 --iocs /home/upgautam/.local/share/mvt/indicators`

##### reading logs
`install_non_market_apps = 1` is a WARNING (not a detection) because it determined to be "no detections" at the end.

         WARNING  [mvt.android.modules.adb.settings] Found suspicious "secure" setting "install_non_market_apps = 1" (enabled installation of non Google Play apps)                          
         INFO     [mvt.android.modules.adb.settings] The Settings module produced no detections!        
This is detection: `WARNING  [mvt.android.modules.adb.root_binaries] Found root binary "su"`

I don't see any other mvt-android functionality is quite useful. Feel free to check more here: https://docs.mvt.re/en/latest/android/adb/

## Indicator of Compromise (IOC)
MVT supports using indicators of compromise (IOCs) to scan mobile devices for potential traces of targeting or infection by known spyware campaigns. 
This includes IOCs published by Amnesty International and other research groups. For example, we have Pegasus IOC here: https://raw.githubusercontent.com/AmnestyTech/investigations/master/2021-07-18_nso/pegasus.stix2

```json

```