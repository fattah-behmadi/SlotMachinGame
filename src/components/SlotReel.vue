<template>
  <div
    class="Reel"
    v-bind:class="{ 'is-locked': locked }"
    v-on:mousedown="lock()"
  >
    <div class="Reel-inner">
      <img class="Reel-image" :src="reelTileData[tile1Index].image" />
    </div>
  </div>
</template>

<script>
export default {
  props: ["value", "canlock"],
  data() {
    return {
      momentum: null,
      audio: {
        spin: new Audio(
          "https://freesound.org/data/previews/120/120373_824230-lq.mp3"
        ),
        spinEnd: new Audio(
          "https://freesound.org/data/previews/145/145441_2615119-lq.mp3"
        ),
        lock: new Audio(
          "https://freesound.org/data/previews/56/56246_91374-lq.mp3"
        ),
      },
      reelTileCount: 15,
      reelTileData: null,
      reelSourceData: [
        {
          name: "Lemon",
          value: 20,
          image: require("@/assets/lemon.jpg"),
        },
        {
          name: "Melon",
          value: 40,
          image: require("@/assets/watermelon.jpg"),
        },
        {
          name: "Orange",
          value: 30,
          image: require("@/assets/orange.jpg"),
        },
        {
          name: "Cherry",
          value: 10,
          image: require("@/assets/cherry.jpg"),
        },
      ],
      reelIndex: 2,
      tile1Index: 0,
      locked: false,
    };
  },

  beforeMount() {
    // create array from random tiles
    this.reelTileData = this.initialReelTile();
  },
  mounted() {
    this.$el.addEventListener("transitionend", this.animateEnd);
  },
  methods: {
    // create array from random tiles
    initialReelTile() {
      let arrayRandomTiles = [];
      this.audio.spin.volume = 0.3;
      this.audio.spinEnd.volume = 0.8;
      this.audio.lock.volume = 0.2;
      this.audio.spin.currentTime = 0.3;

      let reelSourceData = this.reelSourceData.slice(0);
      reelSourceData.sort((a, b) => b.value - a.value);

      reelSourceData.forEach((shape, index) => {
        let times = index * 2 + 1; // 0+1+0=1, 3+2+3=8
        //   let times = index  + 1+index; // 0+1+0=1, 3+2+3=8
        while (times > 0) {
          arrayRandomTiles.push(shape);
          times--;
        }
      });

      return this.shuffle(arrayRandomTiles);
    },
    // shuffle tiles in array
    shuffle(array) {
      let currentIndex = array.length;
      let temporaryValue;
      let randomIndex;

      // While there remain elements to shuffle...
      while (0 !== currentIndex) {
        // Pick a remaining element...
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex -= 1;
        // And swap it with the current element.
        temporaryValue = array[currentIndex];
        array[currentIndex] = array[randomIndex];
        array[randomIndex] = temporaryValue;
      }

      return array;
    },

    // start spin tiles
    run() {
      if (this.locked) {
        this.locked = false;
        this.$emit("stopped", this.reelTileData[this.tile1Index], true);
      } else {
        const min = 8;
        const max = 28;
        const momentum = Math.floor(Math.random() * (max - min + 1) + min);
        this.momentum = momentum;
        this.audio.spin.play();
        this.animate();
      }
    },
    animate() {
      this.$el.classList.add("move");
    },
    animateEnd() {
      this.$el.classList.remove("move");
      this.reIndex();
      this.momentum = this.momentum - 1;
      if (this.momentum > 0) {
        setTimeout(this.animate, 10);
      } else {
        this.$emit("stopped", this.reelTileData[this.tile1Index]);
        this.audio.spinEnd.play();
        this.audio.spin.pause();
        this.audio.spin.currentTime = 0.3;
      }
    },
    reIndex() {
      const end = this.reelTileData.length - 1;
      const index = this.reelIndex === 0 ? end : this.reelIndex - 1;
      this.reelIndex = index;
      this.tile1Index = index === 1 ? end : index === 0 ? end - 1 : index - 2;
    },
    lock() {
      if (this.canlock) {
        this.locked = !this.locked;
        this.audio.lock.play();
      }
    },
  },
};
</script>

<style>
.Reel {
  width: var(--tileSize);
  height: var(--tileSize);
  overflow: hidden;
}
.Reel.is-locked {
  background: rgba(0, 0, 0, 0.1);
}
.Reel:not(:last-child) {
  border-right: 1px solid gray(0, 0.1);
}
.Reel-inner {
  display: flex;
  flex-direction: column;
}
.Reel.move .Reel-inner {
  transition: margin-top 0.04s linear;
  margin-top: var(--tileSize);
}
.Reel-image {
  width: var(--tileSize);
  width: var(--tileSize);
}
</style>
