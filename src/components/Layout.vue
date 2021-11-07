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
      class="user-input"
      type="text"
      v-model="currentInput"
      @keypress.space="isUserInputCorrect"
    />
    <span>1:00</span>
    <button>Restart</button>
  </div>
  <ul>
    <li>Current Input: {{ currentInput }}</li>
    <li>Correct words: {{ rightTotal }}</li>
    <li>Wrong words: {{ wrongTotal }}</li>
  </ul>
</template>

<script setup lang="ts">
import { computed, ComputedRef, ref } from "vue";
const paragraph =
  "Last week we installed a kitty door so that our cat could come and go as she pleases. Unfortunately, we ran into a problem. Our cat was afraid to use the kitty door. We tried pushing her through, and that caused her to be even more afraid. The kitty door was dark, and she couldn’t see what was on the other side. The first step we took in solving this problem was taping the kitty door open. After a couple of days, she was confidently coming and going through the open door. However, when we removed the tape and closed the door, once again, she would not go through. They say you catch more bees with honey, so we decided to use food as bait. We would sit next to the kitty door with a can of wet food and click the top of the can. When kitty came through the closed door, we would open the can and feed her. It took five days of doing this to make her unafraid of using the kitty door. Now we have just one last problem: our kitty controls our lives!".replace(
    /[^a-zA-Z ]/g,
    ""
  );

interface wordItem {
  value: string;
  isCorrect: 1 | 2 | 3;
}

let dividedParagraph: ComputedRef<wordItem[]> = computed(() => {
  return paragraph.split(" ").map((i) => {
    return {
      value: i.toLocaleLowerCase(),
      isCorrect: 1,
    };
  });
});

let currentInput = ref("");
let rightTotal = ref(0);
let wrongTotal = ref(0);
let currentIndex = ref(0);

function isUserInputCorrect() {
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
