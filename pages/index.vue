<template lang="pug">
div
  button(v-if="!running", @click="start") START
  button(v-else, @click="stop") STOP
  div(style="text-align: center; font-size: 5rem")
    TransitionGroup(name="subtitle")
      span(
        style="display: inline-block",
        v-for="c in splitedText",
        :key="c.key"
      ) {{ c.value }}
</template>

<script setup>
let SpeechRecognition = webkitSpeechRecognition || window.SpeechRecognition;
let recognition = new SpeechRecognition();
recognition.lang = "ja-JP";
recognition.interimResults = true;
recognition.continuous = true;
const result = ref("");
recognition.onresult = (event) => {
  result.value = event.results[event.results.length - 1][0].transcript;
  console.log(result.value);
};
recognition.onend = () => {
  running.value && recognition.start();
};
const running = ref(false);
const start = () => {
  running.value = true;
  recognition.start();
};
const stop = () => {
  running.value = false;
  recognition.abort();
};
onUnmounted(() => {
  running.value = false;
  recognition.abort();
});
const cacheText = ref("");
const cacheArray = ref([]);
const splitedText = computed(() => {
  if (cacheText.value === result.value) {
    return cacheArray.value;
  }
  let spliteds = result.value.split("");
  let ret = [];
  let repeated = {};
  for (var i = 0; i < spliteds.length; i++) {
    let currentText = spliteds[i];
    let current = (spliteds[i - 1] ?? "") + currentText;
    if (!repeated[current]) repeated[current] = 0;
    let repeatedCount = ++repeated[current];
    ret[i] = {
      value: currentText,
      key: current + repeatedCount,
    };
  }
  cacheText.value = result.value;
  cacheArray.value = ret;
  return ret;
});
</script>

<style scoped lang="scss">
.subtitle-move,
.subtitle-enter-active,
.subtitle-leave-active {
  transition: all 0.2s ease;
}

.subtitle-leave-active {
  position: absolute;
}

.subtitle-enter-from,
.subtitle-leave-to {
  opacity: 0;
}

.subtitle-enter-from {
  transform: translateY(1em);
}
.subtitle-leave-to {
  transform: translateY(-1em);
}
</style>
