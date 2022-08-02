<template>
  <div class="SlotMachine">
    <div class="SlotMachine-reels">
      <slot-reel
        ref="reel1"
        :canlock="canlock"
        v-on:stopped="reelStopped"
      ></slot-reel>
      <slot-reel
        ref="reel2"
        :canlock="canlock"
        v-on:stopped="reelStopped"
      ></slot-reel>
      <slot-reel
        ref="reel3"
        :canlock="canlock"
        v-on:stopped="reelStopped"
      ></slot-reel>
    </div>

    <div class="SlotMachine-stats">
      <div class="SlotMachine-coin" v-on:mousedown="insertCoin()"></div>
      <div class="SlotMachine-stat is-credit">
        <div class="SlotMachine-statTitle">Credits</div>
        <div class="SlotMachine-statValue">ABT {{ credits.toFixed(2) }}</div>
        <div class="SlotMachine-statSub">spend ABT {{ spend.toFixed(2) }}</div>
      </div>
      <div class="SlotMachine-stat is-win">
        <div class="SlotMachine-statTitle">Won</div>
        <div class="SlotMachine-statValue">ABT {{ win.toFixed(2) }}</div>
      </div>
    </div>
    <div class="SlotMachine-actions">
      <button class="SlotMachine-button is-spin" v-on:mousedown="spin()">
        Play
      </button>
      <div
        class="SlotMachine-button is-win"
        v-bind:class="{ 'has-win': win }"
        v-on:mousedown="takeWin()"
      >
        Take Win
      </div>
    </div>
  </div>
</template>

<script>
import SlotReel from "./SlotReel.vue";
export default {
  components: {
    SlotReel,
  },
  data() {
    return {
      spend: 6,
      credits: 10,
      win: 0,
      resultData: false,
      canlock: true,
      waslocked: false,
      audio: {
        win: new Audio(
          "https://freesound.org/data/previews/387/387232_1474204-lq.mp3"
        ),
        insertCoin: new Audio(
          "https://freesound.org/data/previews/276/276091_5123851-lq.mp3"
        ),
        bigwin: new Audio(
          "https://freesound.org/data/previews/270/270319_5123851-lq.mp3"
        ),
      },
    };
  },
  mounted() {
    window.addEventListener("keydown", this.keydown);
  },
  methods: {
    keydown: function (e) {
      const key = {
        one: 49,
        two: 50,
        three: 51,
        space: 32,
      };
      if (e.which === key.one) {
        this.$refs.reel1.lock();
        e.preventDefault();
      } else if (e.which === key.two) {
        this.$refs.reel2.lock();
        e.preventDefault();
      } else if (e.which === key.three) {
        this.$refs.reel3.lock();
        e.preventDefault();
      } else if (e.which === key.space) {
        this.spin();
        e.preventDefault();
      }
    },
    spin() {
      if (this.credits > 0 && !this.resultData) {
        this.resultData = [];
        // costs 1 credit
        this.credits = this.credits - 1;
        this.$refs.reel1.run();
        this.$refs.reel2.run();
        this.$refs.reel3.run();
      }
    },
    insertCoin: function () {
      this.audio.insertCoin.currentTime = 0;
      this.audio.insertCoin.play();
      this.credits += 1;
      this.spend += 0.5;
    },
    takeWin: function () {
      if (this.win > 0) {
        this.credits += this.win;
        this.win = 0;
      }
    },
    reelStopped(resultData, wasLocked) {
      if (wasLocked) this.waslocked = wasLocked;
      this.resultData.push(resultData);
      if (this.resultData.length === 3) {
        this.checkWin();
        if (this.waslocked) {
          this.waslocked = false;
          this.canlock = false;
        } else {
          this.canlock = true;
        }
      }
    },
    checkWin() {
      if (this.resultData.length === 3) {
        const v1 = this.resultData[0];
        const v2 = this.resultData[1];
        const v3 = this.resultData[2];
        if (v1.name === v2.name && v2.name === v3.name) {
          if (v1.value >= 10) {
            this.audio.bigwin.play();
          } else {
            this.audio.win.play();
          }
          this.win += v1.value;
          this.waslocked = true; // prevent lock after an unlocked win
        }
      }
      this.resultData = false;
    },
  },
};
</script>

<style>
.SlotMachine {
  border-radius: 5px;
}
.SlotMachine-reels {
  display: flex;
  width: fit-content;
  margin: auto;
}

.SlotMachine-stats {
  display: flex;
  justify-content: flex-end;
  padding: 10px 0;
}
.SlotMachine-coin {
  width: 30px;
  height: 38px;
  background: rgba(255, 0, 0, 0.5);
  border: 3px solid rgba(255, 0, 0, 0.5);
  border-radius: 6px;
  margin: 12px 4px;
  cursor: pointer;
}
.SlotMachine-coin:hover {
  background: rgba(255, 0, 0, 0.658);
}
.SlotMachine-stat {
  display: flex;
  flex-direction: column;
  margin-right: 10px;
  flex-grow: 1;
}
.SlotMachine-statTitle {
  font-size: 12px;
  color: gray(60);
}
.SlotMachine-statValue {
  padding: 5px 10px;
  background: rgba(255, 0, 0, 0.15);
  border-radius: 2px;
  border: 1px solid gray(0);
  font-size: 25px;
  text-align: right;
  color: rgba(255, 0, 0, 0.8);
  text-shadow: 0 0 4px rgba(255, 0, 0, 0.5);
}
.SlotMachine-statSub {
  font-size: 9px;
  color: gray(50);
  text-align: right;
}

.SlotMachine-actions {
  padding: 20px 10px;
  display: flex;
  justify-content: flex-end;
  background: gray(20);
  border-radius: 0 0 4px 4px;
}
.SlotMachine-button {
  font-size: 17px;
  font-weight: bold;
  padding: 14px 12px;
  margin-left: 20px;
  border-radius: 5px;
  box-shadow: 0 1px 2px 2px gray(0);
}
.SlotMachine-button:focus {
  outline: none;
}
.SlotMachine-button:active {
  box-shadow: 0 0 2px 1px gray(0);
}

.SlotMachine-button.is-spin {
  background: rgba(0, 255, 0, 0.4);
  border: 1px solid rgba(0, 255, 0, 0.4);
}
.SlotMachine-button.is-spin:hover {
  background: rgba(0, 255, 0, 0.43);
}
.SlotMachine-button.is-spin:active {
  background: rgba(0, 255, 0, 0.46);
}

.SlotMachine-button.is-win {
  background: rgba(255, 0, 0, 0.4);
  border: 1px solid rgba(255, 0, 0, 0.5);
}
.SlotMachine-button.is-win.has-win {
  background: rgba(255, 0, 0, 0.99);
}
.SlotMachine-button.is-win:hover {
  background: rgba(255, 0, 0, 0.65);
}
.SlotMachine-button.is-win:active {
  background: rgba(255, 0, 0, 0.7);
}
</style>
