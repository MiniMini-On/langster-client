<template>
  <h5>누적 테스트</h5>
  <Roller class="roller" default-value="0" :value="countOfTest" :isStatic="true" :duration="1000"></Roller>
  <h1>욕냠이 키우기</h1>
  <p>욕냠이는 나쁜말을 냠냠</p>
  <p class="comment">{{ comment }}</p>
  <div class="image-crop"><img alt="평상시 욕냠이" :src="image" /></div>
  <div class="box-frame">
    <div :class="boxClass">
      <span>알려주기 : </span>
      <button id="good" @click="feedback" @mouseover="changeColor" @mouseleave="defaultColor" :class="buttoneClass1">좋은 말</button>
      <button id="bad" @click="feedback" @mouseover="changeColor" @mouseleave="defaultColor" :class="buttoneClass2">나쁜 말</button>
      <button id="pass" @click="feedback" @mouseover="changeColor" @mouseleave="defaultColor" :class="buttoneClass3">넘어가기</button>
    </div>
  </div>
  <form @submit="onSubmit">
    <textarea :disabled="isDisabled" ref="test" v-model="message" @keydown="enter" placeholder="욕냠이에게 말을 걸어보세요&#10;(255자 이내, 욕냠/욕남 사용 불가)" autofocus />
    <button :disabled="isDisabled">↵</button>
  </form>
</template>

<script>
import { Roller } from "vue-roller";
import "vue-roller/dist/style.css";

export default {
  name: "App",
  components: {
    Roller,
  },
  created() {
    this.axios
      .get("api/language/count")
      .then((res) => {
        this.countOfTest = res.data.number.toString();
      })
      .catch(() => {
        alert("서버 점검 중 입니다");
      });
  },
  data() {
    return {
      image: require("./assets/langster1.gif"),
      message: "",
      isDisabled: false,
      countOfTest: "0",
      comment: "배고파 ~ 배고파 ~",
      buttoneClass1: "training",
      buttoneClass2: "training",
      buttoneClass3: "training",
      boxClass: "training-box-hidden",
    };
  },
  methods: {
    onSubmit(e) {
      e.preventDefault();
      if (this.message == 0) {
        return alert("입력된 값이 없습니다.");
      } else if (this.message > 255) {
        return alert("욕냠이가 알아듣기에 너무 긴 메시지 입니다.");
      } else {
        const message = this.message;
        this.message = message;
        const regExp = /[a-zA-Z0-9!?@#$%^&*():;+-=~{}<>_[\]|"',./`₩]/g;
        const trimed = this.message.replace(regExp, "");
        if (trimed == 0) {
          return alert("욕냠이는 한글에 반응합니다.");
        } else if (this.message.includes("욕남") || this.message.includes("욕냠")) {
          return alert("욕냠과 욕남은 테스트의 정확성을 위해 사용하실 수 없습니다 ~");
        } else {
          this.isDisabled = true;
          this.comment = "입력 중 ...";
          this.boxClass = "training-box";
          this.image = require("./assets/loading.gif");
          this.axios
            .post("api/language/test", { input: this.message })
            .then((res) => {
              setTimeout(() => {
                if (res.data.output == 0) {
                  this.image = require("./assets/langster2.gif");
                  this.comment = "좋은 말 고마워 ~";
                } else if (res.data.output == 1) {
                  this.image = require("./assets/langster3.gif");
                  this.comment = "나쁜 말 냠냠 ~";
                }
              }, 1500);
            })
            .catch(() => {
              alert("서버 점검 중 입니다");
            });
        }
      }
    },
    enter(e) {
      if (e.keyCode == 13 && !this.isDisabled) {
        this.onSubmit(e);
      }
    },
    changeColor(e) {
      if (e.target.id == "good") {
        this.buttoneClass1 = "training-color";
      } else if (e.target.id == "bad") {
        this.buttoneClass2 = "training-color";
      } else {
        this.buttoneClass3 = "training-color";
      }
    },
    defaultColor(e) {
      if (e.target.id == "good") {
        this.buttoneClass1 = "training";
      } else if (e.target.id == "bad") {
        this.buttoneClass2 = "training";
      } else {
        this.buttoneClass3 = "training";
      }
    },
    clear() {
      this.image = require("./assets/langster1.gif");
      this.isDisabled = false;
      this.message = "";
      this.comment = "배고파 ~ 배고파 ~";
      this.boxClass = "training-box-hidden";
    },
    feedback(e) {
      if (e.target.id == "pass") {
        this.clear();
      } else if (e.target.id == "good") {
        if (this.comment != "좋은 말 고마워 ~") {
          this.axios.post("api/language/training", { content: this.message, is_bad: false }).then(() => {
            this.clear();
          });
        } else {
          this.clear();
        }
      } else {
        if (this.comment == "좋은 말 고마워 ~") {
          this.axios.post("api/language/training", { content: this.message, is_bad: true }).then(() => {
            this.clear();
          });
        } else {
          this.clear();
        }
      }
    },
  },
};
</script>

<style>
@font-face {
  font-family: "SBAggroB";
  src: url("https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2108@1.1/SBAggroB.woff") format("woff");
  font-weight: normal;
  font-style: normal;
}
@font-face {
  font-family: "omyu_pretty";
  src: url("https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2304-01@1.0/omyu_pretty.woff2") format("woff2");
  font-weight: normal;
  font-style: normal;
}
#app {
  font-family: SBAggroB, Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  align-items: center;
  color: black;
  margin-top: 5vh;
  overflow-x: hidden;
  height: 100vh;
}

h1 {
  font-size: 44px;
  margin: 0;
}
h5 {
  margin: 0;
}
form {
  text-align: center;
  height: 112px;
  line-height: 110px;
}
p {
  font-size: 20px;
  margin: 0;
}
.comment {
  position: relative;
  top: 50px;
  z-index: 10;
  color: #f8725d;
}
img {
  position: relative;
  width: 350px;
}

/* .submit-box {
  height: 112px;
  line-height: 110px;
} */

textarea {
  width: 230px;
  height: 60px;
  font-size: 18px;
  line-height: 20px;
  border: 2px solid black;
  padding: 10px;
  color: black;
  background-color: peachpuff;
  font-family: "omyu_pretty";
  resize: none;
  vertical-align: middle;
}

.image-crop {
  height: 300px;
  overflow: hidden;
}

button {
  padding: 0;
  border: 2px solid black;
  font-family: "omyu_pretty";
  font-size: 25px;
  width: 40px;
  height: 83px;
  cursor: pointer;
  vertical-align: middle;
}

.roller {
  width: 50px;
  margin: 0 auto;
  height: 30px;
  font-size: 20px;
  text-align: center;
  justify-content: space-around;
}
.box-frame {
  height: 30px;
}
.training-box {
  position: relative;
  width: 320px;
  height: 130px;
  margin: 0 auto;
  font-size: 15px;
  background-color: beige;
  z-index: 10;
  background-color: rgba(249, 157, 157, 0.645);
}
.training-box-hidden {
  display: none;
}

.training {
  font-size: 15px;
  font-family: "SBAggroB";
  width: 70px;
  border: 2px solid black;
  border-radius: 10px;
  height: 30px;
  margin: 3px;
}

.training-color {
  font-size: 15px;
  font-family: "SBAggroB";
  width: 70px;
  border: 2px solid black;
  border-radius: 10px;
  height: 30px;
  margin: 3px;
  background-color: #f89e90;
}
</style>
