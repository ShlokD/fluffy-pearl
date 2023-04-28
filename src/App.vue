<script lang="ts">
import { get, set } from "idb-keyval";
import { defineComponent } from "vue";

export default defineComponent({
  data() {
    return {
      messages: [] as Array<string>,
      optionState: "BASE",
      voices: [] as Array<SpeechSynthesisVoice>,
      selectedVoice: 0,
      text: "",
    };
  },
  mounted() {
    function getVoices(): Promise<SpeechSynthesisVoice[]> {
      return new Promise(function (resolve) {
        const synth = window.speechSynthesis;
        let id = -1;

        id = setInterval(() => {
          if (synth.getVoices().length !== 0) {
            resolve(synth.getVoices());
            clearInterval(id);
          }
        }, 10);
      });
    }

    const write = async () => {
      const val = await get("messages");
      if (!val) {
        this.messages = [
          "I am hungry",
          "I am thirsty",
          "I need to go to the bathroom",
        ];
        set("messages", JSON.stringify(this.messages));
      } else {
        this.messages = JSON.parse(val);
      }
      this.voices = await getVoices();
    };
    write();
  },
  methods: {
    greet(msg: string) {
      const utterThis = new SpeechSynthesisUtterance(msg);
      utterThis.rate = 0.6;
      utterThis.voice = this.voices[this.selectedVoice];
      utterThis.lang = "en-US";
      window.speechSynthesis.speak(utterThis);
    },
    onEdit() {
      this.optionState = "EDIT";
    },
    onSettings() {
      this.optionState = "SETTINGS";
    },
    onBase() {
      this.optionState = "BASE";
    },
    onVoiceChange(index:string) {
      this.selectedVoice = Number(index);
    },
    addText() {
      if (this.text.length > 0) {
        this.messages.push(this.text);
        set("messages", JSON.stringify(this.messages));
      }
      this.text = "";
    },
  },
});
</script>

<template>
  <main class="flex flex-col w-full h-full items-center m-2">
    <div
      className="flex flex-col items-center border border-green-800 border-4 border-dotted rounded-lg p-4 my-4 w-11/12"
      :style="{ minHeight: '400px' }"
    >
      <h1 className="font-bold text-2xl mx-2 text-green-800 ">Speech Reader</h1>

      <div className="flex flex-col items-center justify-center w-full">
        <img v-if="optionState === 'BASE'" src="/reflecting.png" alt="" />
        <div
          className="w-full my-4 flex items-center justify-center"
          v-if="optionState === 'EDIT'"
        >
          <input
            v-model="text"
            className="p-4 border border-green-900 mx-2 w-9/12"
            placeholder="Enter text..."
            aria-label="Enter text"
          />
          <button
            className="bg-green-200 text-green-900 p-4 rounded-lg"
            @click="addText()"
          >
            Add
          </button>
        </div>
        <div
          className="w-full flex justify-center items-center"
          v-if="optionState === 'SETTINGS'"
        >
          <select
            className="w-full bg-green-200 text-green-900 my-2 p-4 lg:text-lg"
            @change="onVoiceChange(($event?.target as HTMLSelectElement)?.value)"
            aria-label="voices"
          >
            <option
              v-for="(voice, index) in voices"
              :value="index"
              :key="index"
              :selected="voice.name === voices[selectedVoice].name"
            >
              {{ voice.name }}
            </option>
          </select>
        </div>
        <button
          className="bg-green-200 text-green-900 font-bold text-lg p-4 rounded-lg"
          v-if="optionState === 'SETTINGS' || optionState === 'EDIT'"
          @click="onBase()"
        >
          Close
        </button>
      </div>
    </div>
    <div className="flex items-center justify-around my-2 w-full">
      <button
        @click="onEdit()"
        className="bg-green-200 rounded-full p-4 my-4"
        aria-label="Add Text"
      >
        <img src="/edit.png" alt="" height="32" width="32" />
      </button>
      <button
        @click="onSettings()"
        className="bg-green-200 rounded-full p-4 my-4"
        aria-label="Settings"
      >
        <img src="/settings.png" alt="" height="32" width="32" />
      </button>
    </div>
    <div class="flex flex-wrap my-4 items-stretch justify-center w-full h-auto">
      <button
        className="w-6/12 lg:w-3/12 bg-green-200 text-green-900 rounded-lg p-4 mx-2 my-2 font-bold text-lg"
        v-for="(message, index) in messages"
        :key="index"
        @click="greet(message)"
      >
        {{ message.toUpperCase() }}
      </button>
    </div>
  </main>
</template>
