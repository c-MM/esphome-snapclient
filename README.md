# esphome-snapclient

This is a temporary repo to build an esphome based snapclient for the 
esp32-louder platform from Andriy Malyshenko (@anabolyc)

It is just for easy build in the esphome web interface.

It is based on the works from:
 - @CarlosDerSeher [esp snapclient](https://github.com/CarlosDerSeher/snapclient)
 - @luar123 [esphome snapclient](https://github.com/esphome/esphome/pull/8350)
 - @mrtoy-me [tas5805m driver](https://github.com/mrtoy-me/esphome-components-test)
 - @anabolyc [esp32 audio hardware](https://github.com/sonocotta/esp32-audio-dock)


example yaml:
```
substitutions:
  name: esphome-web-dc73f8
  friendly_name: louder-esp32
  task_stack_in_psram: "false"

esphome:
  name: ${name}
  friendly_name: ${friendly_name}
  min_version: 2024.11.0
  name_add_mac_suffix: false
  platformio_options:
    board_build.flash_mode: dio
  on_boot:
    priority: -100
    then:
      - lambda: id(tas5805m_dac).refresh_eq_gains();

esp32:
  board: mhetesp32minikit
  flash_size: 8MB
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

external_components:
  - source: github://mrtoy-me/esphome-components-test@eq_dev
    components: [ tas5805m ]
    refresh: 0s
  - source: github://c-MM/esphome-snapclient
    components: [ snapclient ]
    refresh: 0s

network:
  enable_ipv6: true

psram:
  speed: 80MHz

globals:
  - id: current_volume
    type: float
  - id: eq_gain_refreshed
    type: bool
    initial_value: "false"

i2c:
  sda: GPIO21
  scl: GPIO27
  frequency: 400kHz
  scan: True

audio_dac:
  - platform: tas5805m
    id: tas5805m_dac
    enable_pin: GPIO33
    analog_gain: -5db

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

switch:
  - platform: template
    name: Enable
    id: enable_louder
    optimistic: true
    restore_mode: ALWAYS_ON
    turn_on_action:
      lambda: id(tas5805m_dac).set_deep_sleep_off();
    turn_off_action:
      lambda: id(tas5805m_dac).set_deep_sleep_on();

  - platform: template
    name: Enable EQ Control
    id: enable_eq_control
    optimistic: true
    restore_mode: RESTORE_DEFAULT_ON
    turn_on_action:
      - lambda: id(tas5805m_dac).set_eq_on();
    turn_off_action:
      - lambda: id(tas5805m_dac).set_eq_off();
```
