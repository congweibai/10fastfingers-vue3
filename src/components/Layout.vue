<template>
  <div class="paragraph-container">
    <span v-for="(word, index) in dividedParagraph" :key="index">
      <span
        :class="[
          { 'highlight-index': currentIndex === index },
          { isCorrect: word.isCorrect === 2 },
          { isWrong: word.isCorrect === 3 },
          { 'highlight-error': hightErrorIndex === index },
        ]"
        >{{ word.value }}</span
      >
      <span v-text="' '"></span>
    </span>
  </div>
  <div class="input-containter">
    <input
      :disabled="isDisabled"
      class="user-input"
      type="text"
      v-model="currentInput"
      @keypress.space="isUserInputCorrect"
    />
    <span> {{ timeLeft }}s </span>
    <button @click="restartCount">Restart</button>
  </div>
  <div v-if="!showResult">Current Input: {{ currentInput }}</div>
  <ul class="result-container">
    <li>Results:</li>
    <li>
      <div>{{ rightTotal + wrongTotal }} WPM</div>
      <div style="font-size: 10px">(words per minute)</div>
    </li>
    <li>
      Accuracy:
      {{ accuracy === "NaN" ? 0 : accuracy }}%
    </li>
    <li>Correct words: {{ rightTotal }}</li>
    <li>Wrong words: {{ wrongTotal }}</li>
  </ul>
</template>

<script setup lang="ts">
import { Ref, ref, onMounted, computed, watch } from "vue";

interface wordItem {
  value: string;
  isCorrect: 1 | 2 | 3;
}

declare global {
  interface Window {
    txtgen?: any;
  }
}

let dividedParagraph: Ref<wordItem[]> = ref([]);

let currentInput = ref("");
let hightErrorIndex = ref(-1);
watch(
  () => currentInput.value,
  (newValue) => {
    newValue = newValue.trim();
    if (!newValue) {
      hightErrorIndex.value = -1;
      return;
    }
    const matchKeyRegex = new RegExp("^" + newValue, "g");
    const wordItem = dividedParagraph.value[currentIndex.value].value;
    if (!matchKeyRegex.test(wordItem)) {
      hightErrorIndex.value = currentIndex.value;
    } else {
      hightErrorIndex.value = -1;
    }
  }
);
let rightTotal = ref(0);
let wrongTotal = ref(0);
let currentIndex = ref(0);
let screenTimer = ref();
let accuracy = computed(() => {
  return (
    (rightTotal.value / (rightTotal.value + wrongTotal.value)) *
    100
  ).toFixed(2);
});

function isUserInputCorrect() {
  if (currentIndex.value === 0) {
    screenTimer.value = setupTimer();
  }
  const targetValue = dividedParagraph.value[currentIndex.value].value.trim();
  const userInputValue = currentInput.value.trim();

  if (targetValue === userInputValue) {
    rightTotal.value += 1;
    dividedParagraph.value[currentIndex.value].isCorrect = 2;
  } else {
    wrongTotal.value += 1;
    dividedParagraph.value[currentIndex.value].isCorrect = 3;
  }

  currentInput.value = "";
  if (currentIndex.value <= dividedParagraph.value.length - 2)
    currentIndex.value += 1;
}

let timeLeft = ref(60);
let isDisabled = ref(false);
let showResult = ref(false);
function setupTimer() {
  return setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value -= 1;
    } else {
      isDisabled.value = true;
      showResult.value = true;
    }
  }, 1000);
}

async function restartCount() {
  currentIndex.value = 0;
  // const result = await axios.get("http://metaphorpsum.com/paragraphs/1/25");
  let txtgen = window.txtgen;
  const paragraph = (txtgen.paragraph([30]) || "").replace(/[^a-zA-Z ]/g, "");
  dividedParagraph.value = paragraph.split(" ").map((i: string) => {
    return {
      value: i.toLocaleLowerCase(),
      isCorrect: 1,
    };
  });
  isDisabled.value = false;
  timeLeft.value = 60;
  rightTotal.value = 0;
  wrongTotal.value = 0;
  if (screenTimer.value) {
    clearInterval(screenTimer.value);
    screenTimer.value = null;
  }
}

onMounted(() => {
  restartCount();
});
</script>

<style lang="scss" scoped>
.paragraph-container {
  padding: 10px;
  background: white;
  border-radius: 5px;
  border: 2px solid black;
  font-weight: bold;
  font-size: 1.5rem;
}

.input-containter {
  margin-top: 15px;
  display: flex;
  align-items: center;
  .user-input {
    width: 100%;
    height: 30px;
  }
}

.highlight-index {
  border-radius: 2px;
  background: lightblue;
}
.highlight-error {
  background: red;
}
.isCorrect {
  color: green;
}
.isWrong {
  color: red;
}
</style>
