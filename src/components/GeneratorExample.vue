<template>
  <div class="container">
    <div class="jumbotron title-box">
      <h1 class="mytitle engrave">Async Generator Progress Bar
      </h1>
      <hr />
      <div class="progress container mybar-container" style="width:80%">
        <div id="bar" class="progress-bar mybar" style="width:0%">
          {{ precise(state.loaded, 4) }}%
        </div>
      </div>
      <div id='msg' class="engrave" />
      <div id="loadmsg" class="engrave" hidden="true">
        Loading... please wait.
        <span id="spinspan-small" class="spinner-border spinner-border-sm text-success"></span>
      </div>
    </div>
  </div>
</template>

<script setup>
window.onload = () => {
  document.documentElement.style.setProperty('--percentage', state.loaded);
};

import { defineProps, onMounted, reactive, resolveDirective } from 'vue'
defineProps({
  // msg: String
})

const precise = (x, y) => Number.parseFloat(x).toPrecision(y);

onMounted(hook => {
  // call async generation function.
  document.getElementById('loadmsg').hidden = false;
  gen();
});

const gen = () => {
  let count   = state.max;
  let msg     = document.getElementById('msg');

  async function* progress() {
    let total = 0;
    while (--count) {
      // mimic multiple requests to a server
      const delay =  Math.round(500 * Math.random());
      await new Promise(resolve => setTimeout(resolve, delay));
      total += delay;

      yield { count: count, took: `${delay / 1000} s` };
    }

    // 1 sec pause before finish
    await new Promise(resolve => setTimeout(resolve, 1000));

    msg.innerHTML = `Done: loaded ${state.max} items in ${total / 1000}s`;
    state.loaded  = 100;

    document.getElementById('bar').style.width = `${state.loaded}%`;
    document.getElementById('loadmsg').hidden  = true;
  }
  const progressbar = progress();
  (async () => {
    for await (const val of progressbar) {
      state.loaded = (state.max - val.count) / state.max * 100;
      document.documentElement.style.setProperty('--pct-color', getRgb(state.loaded));
      document.documentElement.style.setProperty('--percentage', state.loaded);
      const bar = document.getElementById('bar');
      bar.style.width = `${state.loaded}%`;
      msg.innerHTML   = `count:  ${state.max - val.count} - took ${precise(val.took, 4)}`;
      // console.log(msg.innerHTML);
    }
  })();
}

const getRgb = percent => {
  const op = 0.8;
  const changeColor = 100 / 6;
  if (percent < changeColor) {
    let pct = percent / changeColor * 100;
    let grn = 140 / 100 * pct;
    return `rgba(255, ${Math.round(grn)}, 0, ${op})`;
  }
  if (percent > changeColor && percent <= (changeColor * 2)) {
    let pct = (percent - changeColor) / changeColor * 100;
    let grn = 140 + (115 / 100 * pct )
    return `rgba(255, ${Math.round(grn)}, 0, ${op})`;
  }
  if (percent > (changeColor * 2) && percent <= changeColor * 3) {
    let pct = (percent - changeColor * 2) / changeColor * 100;
    let red = 255 - (255 / 100 * pct )
    let grn = 255 - (127 / 100 * pct )
    return `rgba(${Math.round(red)}, ${Math.round(grn)}, 0, ${op})`;
    //from RGB(255, 255, 0) to RGB(0, 128, 0)
  }
  if (percent > (changeColor * 3) && percent <= changeColor * 4) {
    let pct = (percent - changeColor * 3) / changeColor * 100;
    let grn = 128 - (128 / 100 * pct )
    let blue = (255 / 100 * pct )
    return `rgba(0, ${Math.round(grn)}, ${Math.round(blue)}, ${op})`;
  }
  if (percent > (changeColor * 4) && percent <= changeColor * 5) {//RGB(148, 0, 211)
    let pct = (percent - changeColor * 4) / changeColor * 100;
    let red =  (148 / 100 * pct )
    let blue = 255 - (44 / 100 * pct )
    return `rgba(${Math.round(red)}, 0, ${Math.round(blue)}, ${op})`;
  }
  if (percent > (changeColor * 5) && percent < changeColor * 6) {
    let pct = (percent - changeColor * 5) / changeColor * 100;
    let red =  148 + (107 / 100 * pct )
    let blue = 211 - (211 / 100 * pct )
    return `rgba(${Math.round(red)}, 0, ${Math.round(blue)}, ${op})`;
  }
  return 'rgb(255,0,0)';
};

const state = reactive({ count: 0, loaded: 0, max: Math.round(Math.random() * 200) })
</script>
<style>
:root {
  --pct-color: rgb(255,255,255);
  --rad: 10px;
  --bar-height: 20px;
  --con-height: 30px;
  --percentage: 0;
}
</style>
<style scoped>

.mybar {
  background: var(--pct-color);
  height: var(--bar-height);
  box-shadow: rgb(18, 18, 46) 0px 50px 100px -20px,
    rgb(51, 27, 27) 0px 30px 60px -30px,
    rgba(20, 63, 11, 0.979) 0px -2px 6px 0px inset;
  border-radius: var(--rad);
  color: rgb(255, 253, 253);
  text-shadow: 1px 1px 1px black;
}

.mybar-container {
  height: var(--con-height);
  box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px inset,
    rgba(0, 0, 0, 0.3) 0px 18px 36px -18px inset;
  padding: 5px;
}

#msg {
  font-weight:bold;
}
.title-box {
  background-color: #e0dcdc;
}
#loadmsg {
  font-weight:bold;
  font-style: italic;
  float: left;
}

.engrave {
  background-color: #666666;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: rgba(255,255,255,0.5) 0px 1px 2px;
}

.mytitle {
  font-weight:bold;
  font-size: 50px;
}
</style>

