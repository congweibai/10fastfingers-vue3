<template>
  <div class="paragraph-container">
    <span v-for="(word, index) in dividedParagraph" :key="index">
      <span
        :class="[
          { 'highlight-index': currentIndex === index },
          { isCorrect: word.isCorrect === 2 },
          { isWrong: word.isCorrect === 3 },
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
import { Ref, ref, onMounted, computed } from "vue";
import axios from "axios";

interface wordItem {
  value: string;
  isCorrect: 1 | 2 | 3;
}

let dividedParagraph: Ref<wordItem[]> = ref([]);

let currentInput = ref("");
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
    dividedParagraph.value[currentIndex.value];
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
  const result = await axios.get("http://metaphorpsum.com/paragraphs/1/25");
  const paragraph = (result.data || "").replace(/[^a-zA-Z ]/g, "");
  dividedParagraph.value = paragraph.split(" ").map((i) => {
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
  console.log("On Mounted");
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
.isCorrect {
  color: green;
}
.isWrong {
  color: red;
}
</style>
