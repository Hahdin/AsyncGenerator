<template>
  <div class="jumbotron">
    <h1>Async Generator Progress Bar</h1>
    <hr />
    <div id="loadmsg" hidden="true">Loading... please wait.</div>
    <div id='msg' />
    <div class="progress container mybar" style="width:80%">
      <span id="spinspan" hidden="true" class="spinner-border spinner-border-sm text-danger"></span>
      <div id="bar" class="progress-bar bg-success" style="width:0%">
        {{ precise(state.loaded, 4) }}%
      </div>
    </div>
  </div>
</template>

<script setup>
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
  let spin    = document.getElementById('spinspan');
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

    msg.innerHTML = 'done';
    spin.hidden   = true;
    state.loaded  = 100;

    document.getElementById('bar').style.width = `${state.loaded}%`;
    document.getElementById('loadmsg').hidden  = true;
  }
  const progressbar = progress();
  (async () => {
    for await (const val of progressbar) {
      state.loaded = (state.max - val.count) / state.max * 100;
      const bar = document.getElementById('bar');
      if (state.loaded >= 50) {
        bar.classList.remove('bg-success');
        bar.classList.add('bg-warning');
      }
      if (state.loaded >= 80) {
        bar.classList.remove('bg-warning');
        bar.classList.add('bg-danger');
      }
      bar.style.width = `${state.loaded}%`;
      msg.innerHTML   = `count:  ${val.count} - took ${precise(val.took, 4)}`;
      // console.log(msg.innerHTML);
    }
  })();
}

const state = reactive({ count: 0, loaded: 0, max: Math.round(Math.random() * 200) })
</script>

<style scoped>
a {
  color: #42b983;
}

.mybar {
  box-shadow: 2px 2px 2px black, -2px -2px 10px rgb(187, 183, 247);
}

#spinspan {
  display: inline;
  position:sticky;
}
</style>