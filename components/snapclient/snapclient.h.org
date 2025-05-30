#pragma once

#ifdef USE_ESP32
#ifndef USE_I2S_LEGACY

#include "esphome/core/defines.h"
#include "esphome/core/component.h"
#include "esphome/components/i2s_audio/i2s_audio.h"
#include "esphome/core/gpio.h"

namespace esphome {
namespace snapclient {

static const char *const TAG = "snapclient";

class SnapClientComponent : public i2s_audio::I2SAudioOut, public Component {
 public:
  void set_mute_pin(GPIOPin *mute_pin) { this->mute_pin_ = mute_pin; }
  void setup() override;
  void loop() override;
  // void dump_config() override;
  float get_setup_priority() const override { return setup_priority::AFTER_WIFI; }
  void set_dout_pin(uint8_t pin) { this->dout_pin_ = pin; }
#ifdef USE_AUDIO_DAC
  void set_audio_dac(audio_dac::AudioDac *audio_dac) { this->audio_dac_ = audio_dac; }
#endif
  bool get_mute_state() { return this->mute_state_; }
  void set_config(std::string name, std::string host, int port) {
    this->name_ = std::move(name);
    this->host_ = std::move(host);
    this->port_ = port;
  }

 protected:
  std::string name_;
  std::string host_;
  int port_;
  float volume_{1.0f};
  bool mute_state_{true};
  QueueHandle_t audio_q_hdl_;
  GPIOPin *mute_pin_{nullptr};
  uint8_t dout_pin_;
#ifdef USE_AUDIO_DAC
  audio_dac::AudioDac *audio_dac_{nullptr};
#endif
};

}  // namespace snapclient
}  // namespace esphome

#endif
#endif
