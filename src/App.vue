<template>
  <main tabindex="-1" @keydown.a="commonButton" @keydown.d="rareButton">
    <h1 style="grid-area: 1/2/2/3; text-align: center">{{ currentWord }}</h1>
    <div style="grid-area: 2/2/3/3" class="common-rare-button-container">
      <button
        class="common-button"
        :class="{ click: clickedCommon, shake: disabledCommon }"
        @click="commonButton"
      >
        Common
      </button>
      <button
        class="rare-button"
        :class="{ click: clickedRare, shake: disabledRare }"
        @click="rareButton"
      >
        Rare
      </button>
    </div>
    <button
      style="grid-area: 3/2/4/3"
      class="save-undo-buttons"
      :class="{ click: clickedUndo, shake: disabledUndo }"
      @click="undo"
    >
      Undo
    </button>
    <button
      style="grid-area: 4/2/5/3"
      class="save-undo-buttons"
      :class="{ click: clickedSave }"
      @click="save"
    >
      Save
    </button>
  </main>
</template>

<script setup lang="ts">
import { onMounted, Ref, ref } from "vue";
import { doc, getDoc, updateDoc } from "firebase/firestore";
import db from "./firestore";
import { useStorage } from "@vueuse/core";
import confetti from "canvas-confetti";

// TODO: change to not test
const allUnlabeledWords = useStorage<string[]>("allUnlabeledWords", []);
const commonWords = useStorage<string[]>("commonWords", []);

const previousWords = useStorage<string[]>("previousWords", []);

const currentWord = ref("");

const clickedCommon = ref(false);
const clickedRare = ref(false);
const clickedUndo = ref(false);
const clickedSave = ref(false);

const disabledCommon = ref(false);
const disabledRare = ref(false);
const disabledUndo = ref(false);

onMounted(async () => {
  if (allUnlabeledWords.value.length === 0 && commonWords.value.length === 0) {
    const docRef = doc(db, "words", "test");
    const docSnap = await getDoc(docRef);

    if (docSnap.exists()) {
      allUnlabeledWords.value = docSnap.data().allUnlabeledWords;
      commonWords.value = docSnap.data().commonWords;
    } else {
      // doc.data() will be undefined in this case
      console.log("No such document!");
    }
  }

  if (allUnlabeledWords.value[allUnlabeledWords.value.length - 1]) {
    currentWord.value =
      allUnlabeledWords.value[allUnlabeledWords.value.length - 1];
  } else {
    currentWord.value = `No words left, your done.`;
  }
});

const commonButton = () => {
  if (allUnlabeledWords.value.length < 1) {
    warnDisabled(disabledCommon);
    return;
  }
  if (clickedCommon.value === true) {
    return;
  }
  clickedCommon.value = true;
  setTimeout(() => {
    clickedCommon.value = false;
  }, 250);
  commonWords.value.push(currentWord.value);
  updateCurrentWord();
};

const rareButton = () => {
  if (allUnlabeledWords.value.length < 1) {
    warnDisabled(disabledRare);
    return;
  }
  if (clickedRare.value === true) {
    return;
  }
  clickedRare.value = true;
  updateCurrentWord();
  setTimeout(() => {
    clickedRare.value = false;
  }, 250);
};

const save = async () => {
  if (clickedSave.value === true) {
    return;
  }
  clickedSave.value = true;
  setTimeout(() => {
    clickedSave.value = false;
  }, 250);

  // TODO: change to not test
  await updateDoc(doc(db, "words", "test"), {
    allUnlabeledWords: allUnlabeledWords.value,
    commonWords: commonWords.value,
  });

  previousWords.value = [];
};

const undo = () => {
  if (previousWords.value.length < 1) {
    warnDisabled(disabledUndo);
    return;
  }

  clickedUndo.value = true;
  setTimeout(() => {
    clickedUndo.value = false;
  }, 250);

  const previousWord = previousWords.value[previousWords.value.length - 1];
  previousWords.value.pop();
  // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
  if (commonWords.value.includes(previousWord)) {
    // eslint-disable-next-line @typescript-eslint/no-non-null-assertion
    commonWords.value.splice(commonWords.value.indexOf(previousWord), 1);
  }
  allUnlabeledWords.value.push(previousWord);
  currentWord.value = previousWord;
};

function warnDisabled(disabled: Ref<boolean>) {
  disabled.value = true;
  setTimeout(() => {
    disabled.value = false;
  }, 1500);
}

function updateCurrentWord() {
  if (allUnlabeledWords.value.length < 1) {
    return;
  }

  const lastWord = allUnlabeledWords.value[allUnlabeledWords.value.length - 1];
  allUnlabeledWords.value.pop();
  previousWords.value.push(lastWord);

  if (allUnlabeledWords.value[allUnlabeledWords.value.length - 1]) {
    currentWord.value =
      allUnlabeledWords.value[allUnlabeledWords.value.length - 1];
  } else {
    currentWord.value = `No words left, your done.`;
    confettiFirework();
  }
}

function confettiFirework() {
  let duration = 15 * 1000;
  let animationEnd = Date.now() + duration;
  let defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 };

  function randomInRange(min: number, max: number) {
    return Math.random() * (max - min) + min;
  }

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  let interval: any = setInterval(function () {
    let timeLeft = animationEnd - Date.now();

    if (timeLeft <= 0) {
      return clearInterval(interval);
    }

    let particleCount = 50 * (timeLeft / duration);
    // since particles fall down, start a bit higher than random
    confetti(
      Object.assign({}, defaults, {
        particleCount,
        origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
      })
    );
    confetti(
      Object.assign({}, defaults, {
        particleCount,
        origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
      })
    );
  }, 250);
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

.shake {
  animation: shake 0.82s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
  transform: translate3d(0, 0, 0);
}

@keyframes shake {
  10%,
  90% {
    transform: translate3d(-2px, 0, 0);
  }

  20%,
  80% {
    transform: translate3d(4px, 0, 0);
  }

  30%,
  50%,
  70% {
    transform: translate3d(-8px, 0, 0);
  }

  40%,
  60% {
    transform: translate3d(8px, 0, 0);
  }
}
</style>
