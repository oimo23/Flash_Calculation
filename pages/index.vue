<template>
  <section>
    <h1 class="title">
      The Flash
      <span class="green">暗算</span>
    </h1>

    <div>
      <div class="numberArea" v-if="answerMode">
        <h3 class="countdown">{{ countDown }}</h3>
        <h2>{{ currentNumber }}</h2>
      </div>

      <div class="settingArea" v-if="settingMode">
        <vs-select v-model="digits" class="selectExample" label="桁数">
          <vs-select-item value="2" text="2"/>
          <vs-select-item value="3" text="3"/>
          <vs-select-item value="4" text="4"/>
        </vs-select>

        <vs-select v-model="questions" class="selectExample" label="数字の出現数">
          <vs-select-item value="5" text="5"/>
          <vs-select-item value="10" text="10"/>
          <vs-select-item value="15" text="15"/>
        </vs-select>

        <vs-select v-model="interval" class="selectExample" label="フラッシュ時間 (秒)">
          <vs-select-item value="0.5" text="0.5"/>
          <vs-select-item value="1" text="1"/>
          <vs-select-item value="2" text="2"/>
        </vs-select>

        <vs-button @click="startFlash" color="success" type="filled">start</vs-button>
      </div>

      <div v-if="answerMode">
        <vs-input class="inputx" placeholder="回答を入力" v-model="userAnswer"/>
        <vs-button @click="commitAnswer" color="primary" type="filled">Answer</vs-button>
      </div>

      <div v-if="resultMode">
        <p>{{ message }}</p>
        <br>
        <p>使われた数字</p>
        <ul class="usedNumbers">
          <li v-for="number in numbers" :key="number.index">{{ number }}</li>
        </ul>

        <vs-button @click="reset" color="primary" type="border">One More</vs-button>
      </div>
    </div>
  </section>
</template>

<script>
import Logo from "~/components/Logo.vue";
import IconLink from "~/components/IconLink.vue";

export default {
  components: {
    Logo,
    IconLink
  },
  data() {
    return {
      currentNumber: null,
      numbers: [],
      digits: 2,
      questions: 5,
      interval: 0.5,
      answer: null,
      userAnswer: null,
      settingMode: true,
      answerMode: false,
      resultMode: false,
      message: null,
      countDown: "?"
    };
  },
  methods: {
    makeRandomNumber(digits) {
      let max = "";

      for (let i = 0; i < digits; i++) {
        max += "9";
      }

      const min = 10 * (digits - 1) + 1;

      const randomNumber = Math.floor(Math.random() * (max - min) + min);

      return randomNumber;
    },
    startFlash() {
      this.setConfig();
      this.settingMode = false;
      this.answerMode = true;

      let cnt = 0;

      this.makeCountDown()
        .then(() => {
          let timer = setInterval(() => {
            if (cnt === 0) this.countDown = null;

            if (this.numbers.length === cnt) {
              clearInterval(timer);
              this.countDown = "?";
            }

            this.currentNumber = this.numbers[cnt];
            cnt++;
          }, this.interval * 1000);
        })
        .catch(err => {
          console.log(err);
        });
    },
    makeCountDown() {
      this.countDown = 3;

      return new Promise((resolve, reject) => {
        let count = setInterval(() => {
          if (this.countDown === 1) {
            clearInterval(count);
            this.countDown = "GO!";

            resolve();
          } else {
            this.countDown--;
          }
        }, 1000);
      });
    },
    setConfig() {
      this.numbers = [];
      this.messsage = null;
      this.resultMode = false;

      for (let i = 0; i < this.questions; i++) {
        let number = this.makeRandomNumber(this.digits);
        this.numbers.push(number);
      }

      this.answer = this.numbers.reduce((m, n) => m + n);
    },
    commitAnswer() {
      this.answerMode = false;
      this.resultMode = true;

      if (this.userAnswer == this.answer) {
        this.message = this.answer;
        this.message += " は正解です!";
      } else {
        this.message = "不正解! 正解は";
        this.message += this.answer;
      }
    },
    reset() {
      this.resultMode = false;
      this.settingMode = true;
      this.userAnswer = null;
    }
  }
};
</script>

<style lang="scss" scoped>
section {
  max-width: 90%;
  margin: 0 auto;
}

section > div {
  min-height: 100vh;
  margin: 0 auto;
}

ul li {
  list-style-type: none;
}

.numberArea {
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;

  h2 {
    font-size: 3rem;
  }
}

.usedNumbers {
  display: flex;
  flex-wrap: wrap;
}

.usedNumbers li {
  margin: 5px;
  font-size: 2rem;
  text-align: center;
}

.countdown {
  color: #999;
}

.vs-button {
  margin: 10px 0;
}
</style>
