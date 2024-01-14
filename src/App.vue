<template>
  <div class="container">
    <div class="words">
      <div>Words to pronounce (1 line for each group of words)</div>
      <textarea v-model="text"/>
    </div>
    <div class="options">
      <div class="delay">
        <label for="delay">Delay between each group of words (in seconds) : </label>
        <input v-model="delay" name="delay" type="number">
      </div>
      <div class="repeat">
        <label for="repeat">How many time every group of words is pronounced : </label>
        <input v-model="repeat" name="repeat" type="number">
      </div>
      <div class="rate">
        <label for="rate">Playback speed : ({{rate}})</label>
        <input v-model="rate" name="rate" type="range" min="0.5" max="2" step="0.1" />
      </div>
      <div class="randomize">
        <label for="randomize">Randomize words</label>
        <input v-model="randomize" name="randomize" type="checkbox" />
      </div>
    </div>
    <button @click="speak">Speak</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      text: '',
      delay: 2,
      repeat: 1,
      rate: 1.0,
      randomize: false,
    }
  },
  mounted() {
    let data = localStorage.getItem('data')
    if (null !== data) {
      let props = JSON.parse(data)
      for (let propKey in props) {
        this[propKey] = props[propKey]
      }
    }
  },
  methods: {
    async speak() {
      let words = this.randomize ? this.text.split("\n").sort((a, b) => 0.5 - Math.random()) : this.text.split("\n")
      this.text = words.join("\n")
      localStorage.setItem('data', JSON.stringify(this.$data))
      let _this = this;

      for (let word of words) {
        for (let i = 0; i < this.repeat; i++) {
          await _this.say(word)
          await _this.timer(this.delay * 1000)
        }
      }
    },
    say(text) {
      return new Promise((resolve, reject) => {
        const msg = new SpeechSynthesisUtterance();
        msg.text = text
        msg.rate = this.rate
        msg.lang = 'zh-CN';
        msg.onend = resolve
        msg.onerror = reject
        msg.voice = window.speechSynthesis.getVoices().find(voice => voice.lang === 'zh-CN');

        window.speechSynthesis.speak(msg);
      })
    },
    timer(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms))
    }
  }
}
</script>

<style>
body{
  margin-top: 30px;
}

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

input {
  width: 75px;
}

input, textarea {
  margin: 20px;
  padding: 10px;
  font-size: 18px;
  border: 2px solid #ccc;
  border-radius: 4px;
}

textarea {
  height: 100%;
  min-height: 250px;
  width: 90%;
}

.words {
  width: 40%;
  min-width: 300px;
  display: flex;
  align-items: center;
  flex-direction: column;
}

.options {
  display: flex;
  flex-direction: column;
  text-align: center;

  > * {
    display: flex;
    align-items: center;
    flex-direction: row;
    justify-content: center;
  }

  .rate {
    gap: 1rem;
    flex-direction: column;
    margin: 1rem 0;

    input {
      margin: 0;
      padding: 0;
      width: 75%;
    }
  }
}

button {
  font-size: 20px;
  margin-top: 15px;
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:hover {
  background-color: #3e8e41;
}
</style>
