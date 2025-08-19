# esphome-snapclient

This is a temporary repo to build an esphome based snapclient for the 
esp32-louder platform from Andriy Malyshenko (@anabolyc)

It is just for easy build in the esphome web interface.

It is based on the works from:
 - @CarlosDerSeher [esp snapclient](https://github.com/CarlosDerSeher/snapclient)
 - @luar123 [esphome snapclient](https://github.com/esphome/esphome/pull/8350)
 - @mrtoy-me [tas5805m driver](https://github.com/mrtoy-me/esphome-tas5805m)
 - @anabolyc [esp32 audio hardware](https://github.com/sonocotta/esp32-audio-dock)


example yaml for louder-esp32:
```
substitutions:
  name: esphome-web-dc73f8
  friendly_name: louder-esp32
  task_stack_in_psram: "false"

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  min_version: 2025.7.0
  name_add_mac_suffix: false
  platformio_options:
    board_build.flash_mode: dio
  
esp32:
  board: mhetesp32minikit
  flash_size: 8MB
  cpu_frequency: 240MHz
  framework:
    type: esp-idf
    version: recommended
    sdkconfig_options:
      CONFIG_BT_ALLOCATION_FROM_SPIRAM_FIRST: "y"
      CONFIG_BT_BLE_DYNAMIC_ENV_MEMORY: "y"
      CONFIG_MBEDTLS_EXTERNAL_MEM_ALLOC: "y"
      CONFIG_MBEDTLS_SSL_PROTO_TLS1_3: "y"  # TLS1.3 support isn't enabled by default in IDF 5.1.5

# Enable logging
logger:
  level: DEBUG

# Enable Home Assistant API
api:

# Allow Over-The-Air updates
ota:
  platform: esphome
  password: !secret esphome_ota_password

wifi:
  ssid: !secret esphome_wifi_ssid
  password: !secret esphome_wifi_password
  ap:
    ssid: "$name Hotspot"
    password: !secret esphome_ap_password

captive_portal:

#ethernet:
#  type: W5500
#  clk_pin: GPIO18
#  mosi_pin: GPIO23
#  miso_pin: GPIO19
#  cs_pin: GPIO05
#  interrupt_pin: GPIO35
#  reset_pin: GPIO14
#  clock_speed: 40MHz

external_components:
  - source: github://mrtoy-me/esphome-tas5805m@main
    components: [ tas5805m ]
    refresh: 0s
  - source: github://c-MM/esphome-snapclient@main
    components: [ snapclient ]
    refresh: 0s

network:
  enable_ipv6: false

psram:
  speed: 80MHz

i2c:
  sda: GPIO21
  scl: GPIO27
  frequency: 400kHz
  scan: True

audio_dac:
  - platform: tas5805m
    id: tas5805m_dac
    enable_pin: GPIO33
    analog_gain: -8.5db
    refresh_eq: BY_SWITCH
    dac_mode: BTL
    mixer_mode: STEREO
    volume_max: 0dB
    volume_min: -50db
    update_interval: 10s

i2s_audio:
  i2s_lrclk_pin: GPIO25
  i2s_bclk_pin: GPIO26

snapclient:
  hostname: !secret snapserver_ip
  # port: 1704  # default
  name: $friendly_name
  # mute_pin: 18
  audio_dac: tas5805m_dac
  i2s_dout_pin: GPIO22
  # webserver_port: 8080  # start webserver for equalizer

switch:
  - platform: tas5805m
    enable_dac:
      name: Enable Louder
      id: enable_louder
      restore_mode: ALWAYS_ON
    enable_eq:
      name: Enable EQ Control
      restore_mode: ALWAYS_OFF

number:
  - platform: tas5805m
    eq_gain_band20Hz:
      name: Gain ---20Hz
    eq_gain_band31.5Hz:
      name: Gain ---31.5Hz
    eq_gain_band50Hz:
      name: Gain ---50Hz
    eq_gain_band80Hz:
      name: Gain ---80Hz
    eq_gain_band125Hz:
      name: Gain --125Hz
    eq_gain_band200Hz:
      name: Gain --200Hz
    eq_gain_band315Hz:
      name: Gain --315Hz
    eq_gain_band500Hz:
      name: Gain --500Hz
    eq_gain_band800Hz:
      name: Gain --800Hz
    eq_gain_band1250Hz:
      name: Gain -1250Hz
    eq_gain_band2000Hz:
      name: Gain -2000Hz
    eq_gain_band3150Hz:
      name: Gain -3150Hz
    eq_gain_band5000Hz:
      name: Gain -5000Hz
    eq_gain_band8000Hz:
      name: Gain -8000Hz
    eq_gain_band16000Hz:
      name: Gain 16000Hz

sensor:
  - platform: tas5805m
    faults_cleared:
      name: "Times Faults Cleared"
    update_interval: 60s

binary_sensor:
  - platform: tas5805m
    have_fault:
      name: Any Faults
    left_channel_dc_fault:
      name: Left Channel DC Fault
    right_channel_dc_fault:
      name: Right Channel DC Fault
    left_channel_over_current:
      name: Left Channel Over Current
    right_channel_over_current:
      name: Right Channel Over Current
    otp_crc_check:
      name: CRC Check Fault
    bq_write_failed:
      name: BQ Write Failure
    clock fault:
      name: I2S Clock Fault
    pcdd_over_voltage:
      name: PCDD Over Voltage
    pcdd_under_voltage:
      name: PCDD Under Voltage
    over_temp_shutdown:
      name: Over Temperature Shutdown Fault
    over_temp_warning:
      name: Over Temperature Warning

```
