<template>
  <div>
    <div class="container top-element">
      <div class="input-section">
        <i @click="toggleHideShow" id="hide-show" class="fas fa-eye"></i>
        <div v-if="show">
          <h2>Enter the number of juz that you want to simulate.</h2>
          <p id="note">Use comma (",") or dash ("-") for multiple juz number</p>
          <form>
            <input
              type="text"
              name="juzNumber"
              v-model="juzNumber"
              placeholder="ex: 1,2,30 or 28-30"
              required
              @keydown.enter.prevent="getJuzData"
            />
            <br />
            <button type="button" class="btn btn-dark btn-md" @click.stop.prevent="getJuzData">Start</button>
          </form>
        </div>
      </div>

      <transition name="fade">
        <div v-if="!isValid">
          <h6>Juz Number is invalid.</h6>
        </div>
        <div class="content" v-if="juz !== null & isValid">
          <div class="card">
            <div class="card-header">
              <strong>Juz {{ juz.number }}</strong>
            </div>
            <div class="card-body">
              <p class="card-text">{{ renderArText(juz.ayahs[currentAyah].text) }}</p>
              <p
                v-if="showInfo"
                class="card-text info"
              >Juz {{ juz.number }} QS. {{ juz.ayahs[currentAyah].surah }} : {{ juz.ayahs[currentAyah].number }}</p>
            </div>
          </div>
        </div>
      </transition>
      <div v-if="isLoading" class="semipolar-spinner">
        <div class="ring"></div>
        <div class="ring"></div>
        <div class="ring"></div>
        <div class="ring"></div>
        <div class="ring"></div>
      </div>
      <div
        class="modal fade"
        id="nextQuestion"
        tabindex="-1"
        role="dialog"
        aria-labelledby="nextQuestionLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog modal-dialog-centered">
          <div class="modal-content">
            <div class="modal-body">
              <p>Next Question?</p>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-danger" data-dismiss="modal">No</button>
              <button
                type="button"
                class="btn btn-dark"
                @click="getNextQuestion"
                data-dismiss="modal"
              >Yes</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="footer">
      <div class="container">
        <div class="navigation">
          <div class="row">
            <div id="left" class="col-md-5 col-5">
              <button type="button" class="btn btn-dark btn-md" @click="getPrevAyah">Previous Ayah</button>
            </div>
            <div id="mid" class="col-md-2 col-2">
              <button type="button" class="btn btn-dark btn-md" @click="toggleShowInfo">
                <i id="mid-icon" class="fas fa-eye-slash"></i>
              </button>
            </div>
            <div id="right" class="col-md-5 col-5">
              <button type="button" class="btn btn-dark btn-md" @click="getNextAyah">Next Ayah</button>
            </div>
          </div>
          <div class="row">
            <div class="col">
              <button
                type="button"
                class="next-question btn btn-dark btn-md"
                data-toggle="modal"
                data-target="#nextQuestion"
              >Next Question</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import QuranService from '@/services/QuranService'
import JuzData from '@/../static/data/JuzData.json'

export default {
  name: 'home',
  data () {
    return {
      juzNumber: null,
      juz: null,
      currentAyah: 0,
      beginningOfJuz: false,
      endOfJuz: false,
      show: true,
      isLoading: false,
      isValid: true,
      showInfo: false
    }
  },
  methods: {
    toggleHideShow () {
      if (this.show) {
        document.getElementById('hide-show').className = 'fas fa-eye-slash'
      } else document.getElementById('hide-show').className = 'fas fa-eye'
      this.show = !this.show
    },
    toggleShowInfo () {
      if (this.showInfo) {
        document.getElementById('mid-icon').className = 'fas fa-eye-slash'
      } else document.getElementById('mid-icon').className = 'fas fa-eye'
      this.showInfo = !this.showInfo
    },
    // async getJuzData () {
    //   this.juz = null
    //   this.isLoading = true
    //   await QuranService.getJuz({
    //     numberOfJuz: this.juzNumber
    //   })
    //     .then(response => {
    //       this.juz = response.data.data
    //       this.getNextQuestion()
    //       this.isLoading = false
    //     })
    //     .catch(error => {
    //       console.log(error)
    //     })
    // },
    getJuzData () {
      if (
        (parseInt(this.juzNumber) <= 0) |
        (parseInt(this.juzNumber) > 30) |
        (this.juzNumber === null) |
        (this.juzNumber === '')
      ) {
        this.isValid = false
      } else {
        this.isValid = true
        this.juz = null
        this.isLoading = true
        this.juz = JuzData.data[parseInt(this.juzNumber) - 1]
        this.isLoading = false
        this.getNextQuestion()
        if (this.currentAyah === 1) this.beginningOfJuz = true
        else if (this.currentAyah === this.juz.ayahs.length - 1) {
          this.endOfJuz = true
        }
      }
      this.toggleHideShow()
    },
    renderArText (text) {
      text = this.removeBasmala(text).split(' ')
      //   if (text.length > 12) {
      //     return text.slice(1, 12).join(' ')
      //   }
      return text.join(' ')
    },
    removeBasmala (text) {
      const ayah = this.juz.ayahs[this.currentAyah]
      //   if (ayah.surah.englishName !== 'Al-Faatiha') {
      //     return text.replace(
      //       '\u0628\u0650\u0633\u0652\u0645\u0650 \u0671\u0644\u0644\u0651\u064e\u0647\u0650 \u0671\u0644\u0631\u0651\u064e\u062d\u0652\u0645\u064e\u0670\u0646\u0650 \u0671\u0644\u0631\u0651\u064e\u062d\u0650\u064a\u0645\u0650',
      //       ''
      //     )
      //   }
      //   return text
      if (ayah.name !== 'Al-Faatiha') {
        return text.replace(JuzData.data[0].ayahs[0].text, '')
      }
      return text
    },
    getPrevAyah () {
      if (!this.beginningOfJuz) {
        this.currentAyah = this.currentAyah - 1
        if (this.currentAyah === 0) this.beginningOfJuz = true
        else this.beginningOfJuz = false
      }
    },
    getNextAyah () {
      if (!this.endOfJuz) {
        this.currentAyah = this.currentAyah + 1
        if (this.currentAyah === this.juz.ayahs.length - 1) {
          this.endOfJuz = true
        } else this.endOfJuz = false
      }
    },
    getNextQuestion () {
      this.currentAyah = Math.floor(Math.random() * this.juz.ayahs.length - 1)
      if (this.currentAyah === 0) this.beginningOfJuz = true
      else if (this.currentAyah === this.juz.ayahs.length - 1) {
        this.endOfJuz = true
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
input {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.card-header {
  font-size: 1.5rem;
}

.card-body {
  font-size: 2rem;
}

.card {
  border: 1px solid rgba(0, 0, 0, 0.125);
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.1);
}

.content {
  text-align: center;
}

.navigation {
  margin: 15px 0 15px 0;
}

.next-question {
  margin-top: 15px;
}

button {
  width: 100%;
}

.input-section {
  margin-bottom: 18px;
}

.top-element {
    padding-top: 48px;
    overflow-y: scroll;
    max-height: 100%;
    padding-bottom: 130px;
}

.top-element::-webkit-scrollbar {
  display: none;
}

.footer {
  width: 100%;
  bottom: 0;
  position: fixed;
  background-color: white;
  text-align: center;
}

.semipolar-spinner,
.semipolar-spinner * {
  margin: auto;
  box-sizing: border-box;
}

.semipolar-spinner {
  height: 65px;
  width: 65px;
  position: relative;
}

.semipolar-spinner .ring {
  border-radius: 50%;
  position: absolute;
  border: calc(65px * 0.05) solid transparent;
  border-top-color: #343a40;
  border-left-color: #343a40;
  animation: semipolar-spinner-animation 2s infinite;
}

.semipolar-spinner .ring:nth-child(1) {
  height: calc(65px - 65px * 0.2 * 0);
  width: calc(65px - 65px * 0.2 * 0);
  top: calc(65px * 0.1 * 0);
  left: calc(65px * 0.1 * 0);
  animation-delay: calc(2000ms * 0.1 * 4);
  z-index: 5;
}

.semipolar-spinner .ring:nth-child(2) {
  height: calc(65px - 65px * 0.2 * 1);
  width: calc(65px - 65px * 0.2 * 1);
  top: calc(65px * 0.1 * 1);
  left: calc(65px * 0.1 * 1);
  animation-delay: calc(2000ms * 0.1 * 3);
  z-index: 4;
}

.semipolar-spinner .ring:nth-child(3) {
  height: calc(65px - 65px * 0.2 * 2);
  width: calc(65px - 65px * 0.2 * 2);
  top: calc(65px * 0.1 * 2);
  left: calc(65px * 0.1 * 2);
  animation-delay: calc(2000ms * 0.1 * 2);
  z-index: 3;
}

.semipolar-spinner .ring:nth-child(4) {
  height: calc(65px - 65px * 0.2 * 3);
  width: calc(65px - 65px * 0.2 * 3);
  top: calc(65px * 0.1 * 3);
  left: calc(65px * 0.1 * 3);
  animation-delay: calc(2000ms * 0.1 * 1);
  z-index: 2;
}

.semipolar-spinner .ring:nth-child(5) {
  height: calc(65px - 65px * 0.2 * 4);
  width: calc(65px - 65px * 0.2 * 4);
  top: calc(65px * 0.1 * 4);
  left: calc(65px * 0.1 * 4);
  animation-delay: calc(2000ms * 0.1 * 0);
  z-index: 1;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease-out;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.modal p {
  margin: 0;
}

@keyframes semipolar-spinner-animation {
  50% {
    transform: rotate(360deg) scale(0.7);
  }
}

.info {
  font-size: 1rem;
  font-style: italic;
}

@media screen and (max-width: 426px) {
  h2 {
    font-size: 1.5rem;
  }

  #note {
    font-size: 0.75rem;
  }

  .card-header {
    font-size: 1rem;
  }

  .card-body {
    font-size: 1.5rem;
  }

  .info {
    font-size: 0.9rem;
  }

  #left {
      padding-right: 3px;
  }

  #mid {
    padding: 0 3px 0 3px;
  }

  #right {
      padding-left: 3px;
  }
}
</style>
