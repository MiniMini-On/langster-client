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
      <button id="pass" @click="feedback" :class="buttoneClass1">넘어가기</button>
      <button id="good" @click="feedback" :class="buttoneClass2">좋은 말</button>
      <button id="bad" @click="feedback" :class="buttoneClass3">나쁜 말</button>
      <br />
      <br />
      <br />
      <p class="info">{{ info }}</p>
    </div>
  </div>
  <form @submit="onSubmit">
    <textarea :disabled="isDisabled" ref="test" v-model="message" @keydown="enter" placeholder="욕냠이에게 말을 걸어보세요&#10;(255자 이내)" autofocus />
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
      infoes: ["방금 입력한 말을 욕냠이에게 알려주세요 ♥", "용냠이가 이미 잘 알고 있다면 넘아가기 클릭", "방금 입력한 문장 = 좋은 말 => 좋은 말 클릭", "방금 입력한 문장 = 나쁜 말 => 나쁜 말 클릭"],
      info: "",
      i: 0,
    };
  },
  methods: {
    // 테스트 문장 제출
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
        } else {
          this.isDisabled = true;
          this.comment = "입력 중 ...";
          this.image = require("./assets/loading.gif");

          this.axios
            .post("api/language/test", { input: this.message })
            .then((res) => {
              setTimeout(() => {
                this.boxClass = "training-box";

                if (res.data.output == 0) {
                  this.image = require("./assets/langster2.gif");
                  this.comment = "좋은 말 고마워 ~";
                  this.info = this.infoes[0];
                  this.buttonColorChange();
                } else if (res.data.output == 1) {
                  this.image = require("./assets/langster3.gif");
                  this.comment = "나쁜 말 냠냠 ~";
                  this.info = this.infoes[0];
                  this.buttonColorChange();
                }
              }, 1500);
            })
            .catch(() => {
              alert("서버 점검 중 입니다");
            });
        }
      }
    },
    // 테스트 문장 제출 후 피드백 박스 나타내고, 내부 버튼 색 돌아가며 바뀜

    buttonColorChange() {
      let i = 0;
      this.interval = setInterval(() => {
        if (i == 3) {
          i = 0;
        } else {
          i += 1;
        }
        this.info = this.infoes[i];
        if (i == 1) {
          this.buttoneClass1 = "training-color";
        } else if (i == 2) {
          this.buttoneClass2 = "training-color";
          this.buttoneClass1 = "training";
        } else if (i == 3) {
          this.buttoneClass3 = "training-color";
          this.buttoneClass2 = "training";
        } else if (i == 0) {
          this.buttoneClass3 = "training";
        }
      }, 2000);
    },
    // 엔터 클릭으로 테스트 문장 제출 ( = textarea 엔터로 줄바꿈 방지)
    enter(e) {
      if (e.keyCode == 13 && !this.isDisabled) {
        this.onSubmit(e);
      }
    },

    // 테스트 문장 제출 후 기본 상태로 되돌리기
    clear() {
      clearInterval(this.interval);
      this.image = require("./assets/langster1.gif");
      this.isDisabled = false;
      this.message = "";
      this.comment = "배고파 ~ 배고파 ~";
      this.buttoneClass1 = "training";
      this.buttoneClass2 = "training";
      this.buttoneClass3 = "training";
      this.boxClass = "training-box-hidden";
    },

    // 테스트 문장 응답에 대한 피드백 제출
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
  background-color: rgba(250, 209, 166, 0.8);
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
  line-height: 30px;
  margin: 3px;
}

.training-color {
  font-size: 15px;
  font-family: "SBAggroB";
  width: 70px;
  border: 2px solid black;
  border-radius: 10px;
  height: 30px;
  line-height: 30px;
  margin: 3px;
  background-color: #f89e90;
}

.info {
  font-size: 15px;
  width: 90%;
  margin: 0 5%;
  color: black;
  background-color: rgb(250, 209, 166);
}
</style>
