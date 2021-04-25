<template>
  <v-container>
    <v-row>
      <v-col>
        <v-textarea
          outlined
          title="文本框"
          label="请输入文本"
          v-model="text"
          rows="8"
          autofocus
          :loading="speaking ? 'warning' : false"
        ></v-textarea>
      </v-col>
      <v-col cols="12" class="pt-0">
        <v-row>
          <v-col cols="8">
            <v-row>
              <v-col class="pb-1">
                <v-select
                  dense
                  label="请选择语音"
                  outlined
                  v-model="selectedVoiceItem"
                  :items="voiceList"
                ></v-select>
              </v-col>
            </v-row>
            <v-row>
              <v-col class="pt-0" v-for="ex in [rate, pitch]" :key="ex.label">
                <v-slider
                  class="pr-5"
                  v-model="ex.val"
                  :label="ex.label"
                  persistent-hint
                  :hint="ex.hint"
                  thumb-label
                  thumb-size="24"
                  tick-size="4"
                  ticks
                  track-fill-color="success"
                  max="2"
                  min="0"
                  step="0.1"
                >
                  <template v-slot:append>
                    <span style="width: 5px">{{ ex.val }}</span>
                  </template>
                </v-slider>
              </v-col>
            </v-row>
          </v-col>
          <v-col cols="4">
            <v-btn
              :color="speaking ? 'red' : 'primary'"
              outlined
              class="py-5 mb-5"
              block
              @click="speaking ? stopSpeak() : speak()"
              >{{ speaking ? "停止" : "试听" }}</v-btn
            >
            <v-btn color="primary" class="py-5" block @click="record">
              <v-icon v-show="recording" small class="mr-2"
                >mdi-image-filter-tilt-shift mdi-spin</v-icon
              >录制
            </v-btn>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12"><h3>设置</h3></v-col>
      <v-col cols="8">
        <v-text-field  outlined label="存放的此文件夹：" suffix="/wave/" dense></v-text-field>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import aardio from "aardio";
export default {
  data: () => ({
    text: "",
    selectedVoiceItem: "0",
    synth: window.speechSynthesis,
    voiceList: [],
    voices: [],
    utterThis: null,
    rate: { label: "速度", val: 1, hint: "" },
    pitch: { label: "音调", val: 1, hint: "此项可能是无效的" },
    speaking: false,
    recording: false,
  }),
  created() {
    setTimeout(() => {
      this.populateVoiceList();
    }, 200);
  },
  methods: {
    populateVoiceList() {
      // getList, A-Z 排序
      this.voices = this.synth.getVoices().sort(function (a, b) {
        const aname = a.name.toUpperCase(),
          bname = b.name.toUpperCase();
        if (aname < bname) return -1;
        else if (aname == bname) return 0;
        else return +1;
      });

      for (const i in this.voices) {
        this.voiceList.push({
          value: i,
          text: this.voices[i].name + " (" + this.voices[i].lang + ")",
        });
        if (this.voiceList[i].text.indexOf("Xiaoxiao") >= 0) {
          this.selectedVoiceItem = i + "";
        }
      }
    },
    speak() {
      this.speaking = true;
      if (this.synth.speaking) {
        console.error("speechSynthesis.speaking");
        return;
      }

      if (this.text == "") {
        this.text = "请先输入要朗读的文本";
      }
      this.utterThis = new SpeechSynthesisUtterance(this.text);
      this.utterThis.onend = () => {
        // 停止事件
        console.log("SpeechSynthesisUtterance.onend");
        this.speaking = false;
        if (this.recording) {
          this.recording=false;
          // 正在录制，向宿主发送停止录制请求
          aardio.stopRecord();
          
        }
        
      };
      this.utterThis.onerror = function () {
        console.error("SpeechSynthesisUtterance.onerror");
      };
      this.utterThis.voice = this.voices[this.selectedVoiceItem];
      this.utterThis.rate = this.rate.val;
      this.utterThis.pitch = this.pitch.val;
      this.synth.speak(this.utterThis);
    },
    stopSpeak() {
      // 触发停止事件
      this.synth.pause();
      this.synth.cancel();
    },
    record() {
      // 向宿主发送开始录制请求
      aardio.startRecord();
      this.recording=true;
      this.speak();
    },
  },
};
</script>
