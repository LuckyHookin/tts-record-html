<template>
  <v-container>
    <v-row>
      <v-col>
        <v-textarea
          outlined
          title="文本框"
          label="请输入文本"
          v-model="text"
        ></v-textarea>
      </v-col>
      <v-col cols="12" class="pt-0">
        <v-row>
          <v-col>
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
                  v-model="ex.val"
                  :label="ex.label"
                  persistent-hint
                  :hint="ex.val+''"
                  thumb-label
                  thumb-size="24"
                  tick-size="4"
                  ticks
                  track-fill-color="success"
                  max="2"
                  min="0"
                  step="0.1"
                >
                </v-slider>
              </v-col>
            </v-row>
          </v-col>
          <v-col cols="4">
            <v-btn
              color="primary"
              outlined
              class="py-5 mb-5"
              block
              @click="speak"
              >试听</v-btn
            >
            <v-btn color="primary" class="py-5" block>录制</v-btn>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data: () => ({
    text: "",
    selectedVoiceItem: "0",
    synth: window.speechSynthesis,
    voiceList: [],
    voices: [],
    rate: { label: "速度", val: 1 },
    pitch: { label: "音调", val: 1 },
  }),
  created() {
    setTimeout(() => {
      this.populateVoiceList();
    }, 200);
  },
  methods: {
    st() {
      this.voices = this.synth.getVoices().sort(function (a, b) {
        const aname = a.name.toUpperCase(),
          bname = b.name.toUpperCase();
        if (aname < bname) return -1;
        else if (aname == bname) return 0;
        else return +1;
      });
      console.log(this.voices);
    },
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
      if (this.synth.speaking) {
        console.error("speechSynthesis.speaking");
        return;
      }

      if (this.text == "") {
        this.text = "请先输入要朗读的文本";
        return;
      }
      let utterThis = new SpeechSynthesisUtterance(this.text);
      utterThis.onend = function () {
        console.log("SpeechSynthesisUtterance.onend");
      };
      utterThis.onerror = function () {
        console.error("SpeechSynthesisUtterance.onerror");
      };
      utterThis.voice = this.voices[this.selectedVoiceItem];
      utterThis.rate = this.rate.val;
      utterThis.pitch = this.pitch.val;
      this.synth.speak(utterThis);
    },
  },
};
</script>
