<template>
  <div class="jumbotron">
    <h1>Async Generator Progress Bar</h1>
    <hr />
    <div id="loadmsg" hidden="true">Loading... please wait.</div>
    <div class="progress container mybar" style="width:80%">
      <span id="spinspan" hidden="true" class="spinner-border spinner-border-sm text-danger"></span>
      <div id="bar" class="progress-bar bg-success mybar" style="width:0%">
        {{ precise(state.loaded, 4) }}%
      </div>
    </div>
    <div id='msg' />
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
  box-shadow: rgb(18, 18, 46) 0px 50px 100px -20px,
  rgb(51, 27, 27) 0px 30px 60px -30px,
  rgba(20, 63, 11, 0.979) 0px -2px 6px 0px inset;
  border-radius: 5px;
}

#spinspan {
  display: inline;
  position:sticky;
}

#msg {
  font-family: fantasy;
  box-shadow: rgba(50, 50, 93, 0.25) 0px 50px 100px -20px,
  rgba(240, 127, 127, 0.3) 0px 30px 60px -30px,
  rgba(10, 37, 64, 0.35) 0px -2px 6px 0px inset;
  margin: 50px;
  padding: 5px;

}
</style>

