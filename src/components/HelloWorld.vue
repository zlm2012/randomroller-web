<script>

export default {
  name: "HelloWorld",
  data() {
    return {
      isActive: 'conf',
      strLists: [
          "", ""
      ],
      doors: [],
      strListExample: "単語を改行で入れてください\n🍭\n❌\n⛄\n🦄\n🍌\n💩\n👻\n😻\n💵\n🤡\n🦖\n🍎\n😂\n🖕"
    }
  },
  updated() {
    if (this.isActive === 'roll') {
      this.init(true, 1, 1);
    }
  },
  computed: {
    lists: function () {
      return this.strLists.map(x => x.split("\n"))
    }
  },
  beforeUpdate() {
    this.doors = []
  },
  methods: {
    setDoorRef(el) {
      if (el) {
        this.doors.push(el)
      }
    },
    init(firstInit = true, groups = 1, duration = 1) {
      console.log("init", firstInit, groups, duration)
      this.doors.forEach((door, i) => {
        if (firstInit) {
          door.dataset.spinned = '0';
        } else if (door.dataset.spinned === '1') {
          return;
        }

        const boxes = door.querySelector('.boxes');
        const boxesClone = boxes.cloneNode(false);
        const pool = ['❓'];

        if (!firstInit) {
          const arr = [];
          for (let n = 0; n < (groups > 0 ? groups : 1); n++) {
            arr.push(...this.lists[i]);
          }
          pool.push(...this.shuffle(arr));
          console.log(pool);

          boxesClone.addEventListener(
              'transitionstart',
              function () {
                door.dataset.spinned = '1';
                this.querySelectorAll('.slot-box').forEach((box) => {
                  box.style.filter = 'blur(1px)';
                });
              },
              {once: true}
          );

          boxesClone.addEventListener(
              'transitionend',
              function () {
                this.querySelectorAll('.slot-box').forEach((box, index) => {
                  box.style.filter = 'blur(0)';
                  if (index > 0) this.removeChild(box);
                });
              },
              {once: true}
          );
        }

        for (let i = pool.length - 1; i >= 0; i--) {
          const box = document.createElement('div');
          box.classList.add('slot-box');
          box.style.width = door.clientWidth + 'px';
          box.style.height = door.clientHeight + 'px';
          box.textContent = pool[i];
          boxesClone.appendChild(box);
        }
        boxesClone.style.transitionDuration = `${duration > 0 ? duration : 1}s`;
        boxesClone.style.transform = `translateY(-${door.clientHeight * (pool.length - 1)}px)`;
        door.replaceChild(boxesClone, boxes);
      });
    },

    spin: async function() {
      const groups = ~~(20 / Math.min(...this.lists.map(x => x.length)))
      this.init(false, groups, 3);

      for (const door of this.doors) {
        const boxes = door.querySelector('.boxes');
        const duration = parseInt(boxes.style.transitionDuration);
        let igonred = boxes.offsetLeft;
        boxes.style.transform = 'translateY(0)';
        await new Promise(resolve => setTimeout(resolve, duration * 100));
      }
    },

    shuffle([...arr]) {
      let m = arr.length;
      while (m) {
        const i = Math.floor(Math.random() * m--);
        [arr[m], arr[i]] = [arr[i], arr[m]];
      }
      return arr;
    },

    addCol() {
      this.strLists.push("")
    },
    delCol() {
      this.strLists.pop()
    }
  }
}
</script>

<template>
  <div class="columns">
    <div class="tabs is-toggle is-toggle-rounded column is-9" style="margin: 1%">
      <ul>
        <li v-bind:class="{ 'is-active': isActive === 'conf' }"><a v-on:click="isActive = 'conf'">Set</a></li>
        <li v-bind:class="{ 'is-active': isActive === 'roll' }"><a v-on:click="isActive = 'roll'">Roll!</a></li>
      </ul>
    </div>
    <div class="column is-3" v-if="isActive === 'conf'">
      <button class="button is-rounded is-primary" style="margin-top: 5%" v-on:click="addCol">
        <fa icon="plus" />
      </button>
      <button class="button is-rounded is-danger" style="margin-top: 5%" v-on:click="delCol">
        <fa icon="minus" />
      </button>
    </div>
  </div>
  <div class="tab-contents">
    <div class="content" v-bind:class="{ 'is-active': isActive === 'conf' }">
      <div class="columns">
        <div class="column" v-for="(strList, i) in strLists">
          <textarea class="textarea" v-model="strLists[i]" rows="20" :placeholder="strListExample"></textarea>
        </div>
      </div>
    </div>
    <div class="content" v-bind:class="{ 'is-active': isActive === 'roll' }">
      <div class="door" v-for="list in lists" :ref="setDoorRef">
        <div class="boxes">
          <div class="box">?</div>
        </div>
      </div>
      <template >

      </template>
      <div class="buttons">
        <button id="spinner" v-on:click="spin">Play</button>
        <button id="reseter" v-on:click="init">Reset</button>
      </div>
      <div>
        Inspired by Victor Toschi's <a href="https://medium.com/@victortoschi/how-to-create-a-slot-machine-animation-with-css-and-javascript-9073ab9db9ea">Slot Machine</a>
      </div>
    </div>
  </div>
</template>

<style scoped>
.tab-contents .content {
  display: none;
}
.tab-contents .content.is-active {
  background: #0090c0;
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.door {
  background: #fafafa;
  width: 1000px;
  height: 100px;
  overflow: hidden;
  border-radius: 5px;
  margin: 5px;
  text-align: center;
}

.boxes {
  /* transform: translateY(0); */
  transition: transform 1s ease-in-out;
}

.buttons {
  margin: 1rem 0 2rem 0;
}

button {
  cursor: pointer;
  font-size: 1.2rem;
  margin: 0 0.2rem;
  border: none;
  border-radius: 5px;
  padding: 10px 15px;
}

.info {
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
}
</style>
