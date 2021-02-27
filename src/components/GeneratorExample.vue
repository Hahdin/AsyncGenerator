<template>
  <div class="jumbotron">
    <h1 class="mytitle">Async Generator Progress Bar
    </h1>
    <div id="loadmsg" hidden="true">
      Loading... please wait.
      <span id="spinspan-small" hidden="true" class="spinner-border spinner-border-sm text-success"></span>
    </div>
    <hr />
    <div class="progress container mybar-container" style="width:80%">
      <div id="bar" class="progress-bar mybar" style="width:0%">
        {{ precise(state.loaded, 4) }}%
      </div>
    </div>
    <div id='msg' />
  </div>
</template>

<script setup>
window.onload = () => {
  document.documentElement.style.setProperty('--percentage', state.loaded);
};

import { defineProps, onMounted, reactive } from 'vue'
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
  let spin    = document.getElementById('spinspan-small');
  spin.hidden = false;

  async function* progress() {
    while (--count) {
      // mimic multiple requests to a server
      const delay =  Math.round(500 * Math.random());
      await new Promise(resolve => setTimeout(resolve, delay));

      yield { count: count, took: `${delay / 1000} s` };
    }

    // 1 sec pause before finish
    await new Promise(resolve => setTimeout(resolve, 1000));

    msg.innerHTML = `Done: loaded ${state.max} items.`;
    spin.hidden   = true;
    state.loaded  = 100;

    document.getElementById('bar').style.width = `${state.loaded}%`;
    document.getElementById('loadmsg').hidden  = true;
  }
  const progressbar = progress();
  (async () => {
    for await (const val of progressbar) {
      state.loaded = (state.max - val.count) / state.max * 100;
      document.documentElement.style.setProperty('--percentage', state.loaded);
      const bar = document.getElementById('bar');
      bar.style.width = `${state.loaded}%`;
      msg.innerHTML   = `count:  ${val.count} - took ${precise(val.took, 4)}`;
      // console.log(msg.innerHTML);
    }
  })();
}

const state = reactive({ count: 0, loaded: 0, max: Math.round(Math.random() * 200) })
</script>
<style>
:root {
  --rad: 20px;
  --bar-height: 40px;
  --con-height: 50px;
  --percentage: 0;
}
</style>
<style scoped>
a {
  color: #42b983;
}

.mybar {
  background:rgb(calc( 155 - (155 / 100 * var(--percentage))),
    calc(155 / 100 * var(--percentage)),
    0);
  height: var(--bar-height);
  box-shadow: rgb(18, 18, 46) 0px 50px 100px -20px,
    rgb(51, 27, 27) 0px 30px 60px -30px,
    rgba(20, 63, 11, 0.979) 0px -2px 6px 0px inset;
  border-radius: var(--rad);
}

.mybar-container {
  height: var(--con-height);
  box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px inset,
    rgba(0, 0, 0, 0.3) 0px 18px 36px -18px inset;
  padding: 5px;
}

#msg {
  font-family: fantasy;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
    rgba(240, 127, 127, 0.3) 0px 30px 60px -30px,
    rgba(10, 37, 64, 0.35) 0px -2px 6px 0px inset;
  margin: 50px;
  padding: 5px;
}
.mytitle {
  text-shadow: rgba(61, 61, 61, 0.39) 3px 3px 0px;
}

#loadmsg {
  font-style: italic;
}
</style>

