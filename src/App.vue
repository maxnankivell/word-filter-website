<template>
  <main tabindex="-1" @keydown.a="commonButton" @keydown.d="rareButton">
    <h1 style="grid-area: 1/2/2/3">{{ currentWord }}</h1>
    <div style="grid-area: 2/2/3/3" class="common-rare-button-container">
      <button
        class="common-button"
        :class="{ click: clickedCommon }"
        @click="commonButton"
      >
        Common
      </button>
      <button
        class="rare-button"
        :class="{ click: clickedRare }"
        @click="rareButton"
      >
        Rare
      </button>
    </div>
    <button style="grid-area: 3/2/4/3" class="save-undo-buttons" @click="save">
      Undo
    </button>
    <button style="grid-area: 4/2/5/3" class="save-undo-buttons" @click="undo">
      Save
    </button>
  </main>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";

const allUnlabeledWords = ["four", "five", "test"];
const commonWords = [];

const currentWord = ref("");

const clickedCommon = ref(false);
const clickedRare = ref(false);

onMounted(() => {
  if (allUnlabeledWords[allUnlabeledWords.length - 1]) {
    currentWord.value = allUnlabeledWords[allUnlabeledWords.length - 1];
  } else {
    currentWord.value = `No words left, your done.`;
  }
});

const commonButton = () => {
  if (clickedCommon.value === true) {
    return;
  }
  clickedCommon.value = true;
  setTimeout(() => {
    clickedCommon.value = false;
  }, 250);
  commonWords.push(currentWord.value);
  updateCurrentWord();
};

const rareButton = () => {
  if (clickedRare.value === true) {
    return;
  }
  clickedRare.value = true;
  updateCurrentWord();
  setTimeout(() => {
    clickedRare.value = false;
  }, 250);
};

const save = () => {
  //save shit
};

const undo = () => {
  //undo shit
};

function updateCurrentWord() {
  allUnlabeledWords.pop();
  if (allUnlabeledWords[allUnlabeledWords.length - 1]) {
    currentWord.value = allUnlabeledWords[allUnlabeledWords.length - 1];
  } else {
    currentWord.value = `No words left, your done.`;
  }
}
</script>

<style scoped lang="scss">
main {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  justify-items: center;
  align-items: center;
  row-gap: 32px;
  margin: 0 auto;
  height: 100vh;
}

.common-rare-button-container {
  display: flex;
  justify-content: space-between;
  width: 500px;
  gap: 8px;
}

.common-button {
  background-color: green;
  color: $platinum;

  &:hover {
    background-color: lightgreen;
    color: $white;
  }
}

.rare-button {
  background-color: red;
  color: $platinum;

  &:hover {
    background-color: coral;
    color: $white;
  }
}

.save-undo-buttons {
  width: 500px;
}

.click {
  animation: click 0.25s linear 0s 1 forwards;
}

@keyframes click {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(0.8);
  }

  100% {
    transform: scale(1);
  }
}
</style>
