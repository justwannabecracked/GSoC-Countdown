<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import confetti from "canvas-confetti";

const targetDate = new Date(Date.UTC(2025, 2, 24, 18, 0, 0)); // March 24, 2025, 18:00 UTC
const timeLeft = ref("");
const isComplete = ref(false);
const intervalId = ref(null);
const confettiCanvas = ref(null);
const darkMode = ref(false);
const audioRef = ref(null);
const audioSrc = "/country-roads.mp3";

// Local time equivalent for display
const localTargetDateStr = ref(
  targetDate.toLocaleString(undefined, {
    weekday: "short",
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    second: "2-digit",
    timeZoneName: "short",
  })
);

function updateCountdown() {
  const now = Date.now(); // UTC-based
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
        <!-- toggle icons -->
        <!-- (same as before) -->
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
      <!-- 💡 This is the local time version -->
      <p class="text-md mt-3 italic text-gray-500">
        (Your local time: {{ localTargetDateStr }})
      </p>
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
