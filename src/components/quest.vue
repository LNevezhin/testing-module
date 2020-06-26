<template>
  <div class="container"
       v-if="apidata && apidata.length != 0">
    <div>
      <div class="row">
        <div class="col-6 offset-3"><img :src="imageUrl" /></div>
      </div>
      <div class="row">
        <div class="col-6 offset-4 mt-4"></div>
      </div>
      <div>
        <div v-for="(answer, index) in activeAnswers"
             :key="parseInt(index)">
          <div class="row mt-4">
            <div class="col-1 offset-4">
              <div class="badge badge-pill badge-danger">{{ index + 1 }}</div>
            </div>
            <div class="col-3">
              <input :id="parseInt(index)"
                     type="text"
                     class="form-control"
                     v-bind:class="inputStatus[index]"
                     v-model.trim="inputData[index]"
                     v-bind:disabled="inputStatus[index] == 'is-valid'"
                     @input="checkInput(index)"
                     @focus="getFocus(index)"
                     @blur="lostFocus(index)"
                     autocomplete="off"
                     spellcheck="false" />
              <div class="valid-feedback">
                Верно!
              </div>
              <div class="invalid-feedback">
                Введите правильный ответ!
              </div>
            </div>
            <div class="col"
                 v-if="hintStatus[index] == true">
              <button type="
                 button"
                      class="btn btn-secondary"
                      @click="giveLetter(index)">
                ?
              </button>
            </div>
          </div>
        </div>
        <div class="col-1 mb-4 offset-5 mt-3">
          <button id="next"
                  type="button"
                  class="btn btn-secondary"
                  v-bind:class="{ 'sr-only': nextStatusButton == false }"
                  @click="getNextCard()">
            Далее
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "quest",
    data() {
      return {
        activeUrl: "",
        activeAnswers: [],
        inputData: [],
        correctInputLetters: [], // массив с правильно введенными буквами
        inputStatus: [],
        indexInputLetters: [], // индекс текущей буквы в correctInputLetters[]
        hintStatus: [],
        errCount: [],
        delayStopInput: 5000,
        delayFocusLost: 3000,
        nextStatusButton: false,
        timerStopInput: [],
        timerFocusLost: [],
        activeCard: 3, // номер текущей карточки, оно же ключ, для обновления ДОМ
        apidata: {
          baseApiFilesUrl: "https://www.someurl.ru/files/questionCards/image/",
          questionCards: [{
              image: "pic0000.png",
              answers: ["ПоМиДоР", "КаПуСтА", "пЕрЕц", "БаКЛАЖан", "ЧЕСНОК"]
            },
            {
              image: "pic0001.png",
              answers: [
                "Москва",
                "Киев",
                "Минск",
                "Баку",
                "Ереван",
                "Тбилиси",
                "Алматы"
              ]
            },
            {
              image: "pic0002.png",
              answers: [
                "КруГ",
                "КвадраТ",
                "ТреугольниК",
                "РомБ",
                "ТрапециЯ",
                "ЭллипС"
              ]
            },
            {
              image: "pic0003.png",
              answers: ["красныЙ", "белыЙ", "черныЙ"]
            }
          ]
        }
      };
    },

    created() {
      if (sessionStorage.getItem("activeCard") != null) {
        this.activeCard = sessionStorage.getItem("activeCard");
      }
      if (this.apidata && this.apidata.length != 0) this.newCard();
    },

    computed: {
      imageUrl() {
        return require(`@/assets/img/${this.activeUrl}`);
      }
    },


    methods: {
      newCard: function () {
        this.activeUrl = this.apidata.questionCards[this.activeCard].image;
        this.activeAnswers.length = 0;
        this.indexInputLetters.length = 0;
        this.inputStatus.length = 0;
        this.inputData.length = 0;
        this.correctInputLetters.length = 0;
        this.errCount.length = 0;
        this.hintStatus.length = 0;
        this.timerStopInput.length = this.apidata.questionCards[this.activeCard].answers.length;
        this.timerFocusLost.length = this.apidata.questionCards[this.activeCard].answers.length;

        for (
          let i = 0; i < this.apidata.questionCards[this.activeCard].answers.length; i++
        ) {
          this.activeAnswers.push(this.apidata.questionCards[
            this.activeCard
          ].answers[i].toLowerCase());
          this.hintStatus.push(false);
          this.inputStatus.push("");
          this.errCount.push(0);
          this.correctInputLetters.push("");

        }
        setTimeout(() => this.setFocus(), 0);
      },
      checkInput: function (index) {
        this.stopInput(index);
        this.setInputStatus(index, "");
        this.inputData.splice(index, 1, this.inputData[index].toLowerCase());
        this.indexInputLetters.splice(index, 1, (this.inputData[index].length - 1));
        if (
          this.inputData[index][this.indexInputLetters[index]] !=
          this.activeAnswers[index][this.indexInputLetters[index]]
        )
          this.incorrectInput(index);
        else this.correctInput(index);
      },

      correctInput: function (index) {
        this.setInputStatus(index, "");
        this.errCount.splice(index, 1, 0);
        this.correctInputLetters.splice(index, 1, this.inputData[index]);
        this.validCheck(index);
      },

      validCheck: function (index) {
        if (this.inputData[index] == this.activeAnswers[index]) {
          this.setInputStatus(index, "is-valid");
          this.hintStatus.splice(index, 1, false);
          this.setFocus();
        }
      },

      incorrectInput: function (index) {
        this.setInputStatus(index, "is-invalid");
        this.inputData.splice(index, 1, this.correctInputLetters[index]);
        this.errCount.splice(index, 1, (this.errCount[index] + 1));
        if (this.errCount[index] == 3) this.hintStatus.splice(index, 1, true);
      },

      setInputStatus: function (index, status) {
        this.inputStatus.splice(index, 1, status);
      },

      stopInput: function (index) {
        clearTimeout(this.timerStopInput[index]);
        this.timerStopInput.splice(index, 1, setTimeout(() => {
          if (this.inputStatus[index] != "is-valid") {
            this.setInputStatus(index, "is-invalid");
          }
        }, this.delayStopInput));
      },

      getFocus: function (index) {
        this.stopInput(index);
        this.setInputStatus(index, "");
        clearTimeout(this.timerFocusLost[index]);
      },

      lostFocus: function (index) {
        this.timerFocusLost.splice(index, 1, setTimeout(() => {
          if (this.inputStatus[index] != "is-valid")
            this.setInputStatus(index, "is-invalid");
        }, this.delayFocusLost));
      },

      giveLetter: function (index) {
        this.setFocus(index);
        this.inputData.splice(index, 1, (this.inputData[index] +
          this.activeAnswers[index][this.indexInputLetters[index]]));
        this.correctInputLetters.splice(index, 1, this.inputData[index]);
        this.indexInputLetters.splice(index, 1, (this.indexInputLetters[index] + 1));
        this.validCheck(index);
      },

      setFocus: function (item) {
        const id = this.inputStatus.findIndex(elem => elem != "is-valid");

        if (id != -1 && this.inputStatus[item] != "is-valid" && item != null) {
          const input = document.getElementById(item);
          input.focus();
        }
        if (id != -1 && item == null) {
          const input = document.getElementById(id);
          input.focus();
        }
        if (id == -1) {
          this.nextStatusButton = true;
          const input = document.getElementById("next");
          input.focus();
        }
      },

      getNextCard: function () {
        for (
          let i = 0; i < this.apidata.questionCards[this.activeCard].answers.length; i++
        ) {
          clearTimeout(this.timerStopInput[i]);
          clearTimeout(this.timerFocusLost[i]);
        }

        this.activeCard = parseInt(this.activeCard) + 1;
        if (this.activeCard == this.apidata.questionCards.length)
          this.activeCard = 0;
        sessionStorage.setItem("activeCard", this.activeCard);
        this.nextStatusButton = false;

        this.newCard();
      }
    }
  };
</script>

<style scoped>
  html,
  input,
  input:disabled {
    user-select: none;
  }
</style>