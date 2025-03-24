<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import confetti from "canvas-confetti";

const targetDate = new Date(Date.UTC(2025, 2, 24, 18, 0, 0));
const timeLeft = ref("");
const isComplete = ref(false);
const intervalId = ref(null);
const confettiCanvas = ref(null);
const darkMode = ref(false);
const audioRef = ref(null);
const audioSrc = "/country-roads.mp3";

function updateCountdown() {
  const now = Date.now();
  const diff = targetDate.getTime() - now;

  if (diff <= 0) {
    isComplete.value = true;
    clearInterval(intervalId.value);
    launchConfetti();
    return;
  }

  const totalSeconds = Math.floor(diff / 1000);
  const days = Math.floor(totalSeconds / 86400);
  const hours = Math.floor((totalSeconds % 86400) / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;

  timeLeft.value = `${days}d ${hours}h ${minutes}m ${seconds}s`;
}

function launchConfetti() {
  const myConfetti = confetti.create(confettiCanvas.value, { resize: true });
  const duration = 5 * 1000;
  const end = Date.now() + duration;

  (function frame() {
    myConfetti({
      particleCount: 5,
      angle: 60,
      spread: 55,
      origin: { x: 0 },
    });
    myConfetti({
      particleCount: 5,
      angle: 120,
      spread: 55,
      origin: { x: 1 },
    });

    if (Date.now() < end) {
      requestAnimationFrame(frame);
    }
  })();
}

function playMusic() {
  if (audioRef.value) {
    audioRef.value.muted = false;
    audioRef.value.play().catch((e) => console.warn("Audio play blocked:", e));
  }
}

function toggleDarkMode() {
  darkMode.value = !darkMode.value;
}

onMounted(() => {
  playMusic();
  updateCountdown();
  intervalId.value = setInterval(updateCountdown, 1000);
});

onBeforeUnmount(() => {
  clearInterval(intervalId.value);
});

const formattedTime = computed(() => timeLeft.value);
</script>

<template>
  <main
    :class="[
      darkMode ? 'bg-gray-900 text-white' : 'bg-gray-100 text-black',
      'flex items-center justify-center h-screen w-full transition-colors duration-500',
    ]"
  >
    <div class="absolute top-4 right-4">
      <button @click="toggleDarkMode" class="outline-none text-2xl p-3">
        <span v-if="darkMode">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M21.752 15.002A9.72 9.72 0 0 1 18 15.75c-5.385 0-9.75-4.365-9.75-9.75 0-1.33.266-2.597.748-3.752A9.753 9.753 0 0 0 3 11.25C3 16.635 7.365 21 12.75 21a9.753 9.753 0 0 0 9.002-5.998Z"
            />
          </svg>
        </span>
        <span v-else>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M12 3v2.25m6.364.386-1.591 1.591M21 12h-2.25m-.386 6.364-1.591-1.591M12 18.75V21m-4.773-4.227-1.591 1.591M5.25 12H3m4.227-4.773L5.636 5.636M15.75 12a3.75 3.75 0 1 1-7.5 0 3.75 3.75 0 0 1 7.5 0Z"
            />
          </svg>
        </span>
      </button>
    </div>

    <div v-if="!isComplete" class="text-center">
      <div class="m-4">
        <h1 class="text-5xl font-bold m-2">
          <span class="text-[#ea4335] px-1">Google</span>
          <span class="text-[#4285f4] px-1">Summer</span>
          <span class="text-[#34a853] px-1">of</span>
          <span class="text-[#fbbc05] px-1">Code</span>
        </h1>
      </div>
      <h1 class="text-4xl font-bold mb-4">Countdown to March 24, 18:00 UTC</h1>
      <p class="text-4xl font-mono">{{ formattedTime }}</p>
    </div>

    <div v-else class="text-center">
      <h1 class="text-4xl font-bold mb-4 text-green-400">
        🎉 Congratulations! 🎉
      </h1>
      <p class="text-xl">You made it to March 24, 18:00 UTC!</p>
      <canvas
        ref="confettiCanvas"
        class="fixed top-0 left-0 w-full h-full pointer-events-none"
      ></canvas>
      <audio ref="audioRef" :src="audioSrc" autoplay loop muted></audio>
    </div>
  </main>
</template>
