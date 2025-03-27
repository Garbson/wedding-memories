<template>
  <v-app>
    <div
      class="h-screen pb-10 w-full flex align-middle flex-col overflow-y-auto justify-center items-center relative bg-gradient-to-br from-pink-300 via-red-200 to-purple-300 animate-bg px-4"
    >
      <!-- Conteúdo -->
      <div class="text-center text-white animate-fadeIn">
        <p
          class="text-6xl titulo font-vibes bg-gradient-to-r from-blue-500 via-purple-400 to-pink-400 text-transparent bg-clip-text drop-shadow-lg"
        >
          Garbson & Ana
        </p>
        <p class="text-1xl max-w-2xl mx-auto mb-12 drop-shadow-sm leading-relaxed">
          Deixe uma lembrança especial para o nosso grande dia! Pode ser uma foto
          divertida, um vídeo com carinho ou apenas um sorriso sincero. Vamos guardar cada
          gesto com o coração cheio de amor 💕✨
        </p>

        <!-- Botões -->
        <div
          class="flex flex-col sm:flex-row items-center justify-center gap-4 animate-slideUp"
        >
          <button
            @click="selectFile('image')"
            :disabled="loading"
            class="bg-white text-pink-500 hover:bg-pink-500 hover:text-white transition duration-300 px-8 py-3 rounded-full font-semibold flex items-center gap-2"
          >
            <v-icon icon="mdi-camera" />
            {{ loading && uploadType === "image" ? "Enviando..." : "Enviar Foto" }}
          </button>

          <span class="text-white font-bold text-lg">ou</span>

          <button
            @click="selectFile('video')"
            :disabled="loading"
            class="bg-white text-indigo-500 hover:bg-indigo-500 hover:text-white transition duration-300 px-8 py-3 rounded-full font-semibold flex items-center gap-2"
          >
            <v-icon icon="mdi-video" />
            {{ loading && uploadType === "video" ? "Enviando..." : "Enviar Vídeo" }}
          </button>

          <!-- Input oculto com câmera -->
          <input
            ref="fileInput"
            type="file"
            :accept="uploadType === 'image' ? 'image/*' : 'video/*'"
            :capture="uploadType === 'image' ? 'environment' : 'user'"
            class="hidden"
            @change="handleUpload"
          />
        </div>

        <!-- Modal de Preview -->
        <div
          v-if="dialog"
          class="fixed inset-0 bg-black bg-opacity-60 flex items-center justify-center z-50"
          @click.self="dialog = false"
        >
          <div
            class="bg-gradient-to-br from-purple-500 via-pink-400 to-red-300 p-6 rounded-lg shadow-2xl max-w-sm w-full text-center animate-bouquet"
          >
            <img
              v-if="uploadType === 'image'"
              :src="previewUrl"
              alt="Preview da imagem"
              class="rounded-lg mb-4 max-h-[70vh] mx-auto"
            />
            <p class="text-white text-2xl font-vibes leading-snug">
              Obrigado, iremos guardar essa memória no coração
            </p>
            <p class="text-white text-sm mt-2 font-vibes">
              Fique à vontade para mandar mais lembranças
            </p>
          </div>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script setup lang="ts">
import { ref, nextTick } from "vue";

const cloudName = import.meta.env.VITE_CLOUDINARY_CLOUD_NAME;
const uploadPreset = import.meta.env.VITE_CLOUDINARY_UPLOAD_PRESET;

const fileInput = ref<HTMLInputElement | null>(null);
const loading = ref(false);
const uploadType = ref<"image" | "video" | null>(null);
const uploadedUrl = ref<string>();
const previewUrl = ref<string>();
const dialog = ref(false);

const selectFile = async (type: "image" | "video") => {
  uploadType.value = type;
  await nextTick();
  fileInput.value!.value = "";
  fileInput.value?.click();
};

const handleUpload = async (e: Event) => {
  const target = e.target as HTMLInputElement;
  if (!target.files || !target.files[0] || !uploadType.value) return;

  const file = target.files[0];
  previewUrl.value = URL.createObjectURL(file); // mostra preview
  loading.value = true;

  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", uploadPreset);

  const url = `https://api.cloudinary.com/v1_1/${cloudName}/${uploadType.value}/upload`;

  try {
    const res = await fetch(url, {
      method: "POST",
      body: formData,
    });

    const data = await res.json();
    uploadedUrl.value = data.secure_url;
    dialog.value = true;
    previewUrl.value = data.secure_url;
  } catch (err) {
    alert("Erro ao enviar arquivo 😢");
  } finally {
    loading.value = false;
    target.value = ""; // reset do input
  }
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap");

.animate-bg {
  background-size: 400% 400%;
  animation: animateBg 10s ease infinite;
}

.titulo {
  line-height: 2 !important;
}

@keyframes animateBg {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fadeIn {
  animation: fadeIn 2s ease;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(40px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-slideUp {
  animation: slideUp 2s ease;
}

@keyframes bouquetPop {
  0% {
    opacity: 0;
    transform: scale(0.2) rotate(-20deg);
  }

  60% {
    opacity: 1;
    transform: scale(1.1) rotate(10deg);
  }

  100% {
    transform: scale(1) rotate(0deg);
  }
}

.animate-bouquet {
  animation: bouquetPop 2.5s ease-in-out;
}

@keyframes heartPulse {
  0% {
    transform: scale(0.7);
    opacity: 0;
  }

  50% {
    transform: scale(1.3);
    opacity: 1;
  }

  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.animate-heart {
  animation: heartPulse 2s ease-out forwards;
}

@keyframes floatIn {
  0% {
    transform: translateY(100%);
    opacity: 0;
  }

  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

.animate-float {
  animation: floatIn 3s ease;
}

.delay-300 {
  animation-delay: 0.3s;
}

/* Confetes */
.confetti-container {
  position: absolute;
  top: -10px;
  left: 0;
  right: 0;
  bottom: 0;
}

.confetti {
  position: absolute;
  top: -10px;
  width: 10px;
  height: 10px;
  background-color: var(--tw-confetti-color);
  opacity: 0.8;
  animation: fall linear infinite, sway ease-in-out infinite;
}

.confetti:nth-child(5n + 1) {
  --tw-confetti-color: #ff6b6b;
}

.confetti:nth-child(5n + 2) {
  --tw-confetti-color: #ffd93d;
}

.confetti:nth-child(5n + 3) {
  --tw-confetti-color: #6bcb77;
}

.confetti:nth-child(5n + 4) {
  --tw-confetti-color: #4d96ff;
}

.confetti:nth-child(5n + 5) {
  --tw-confetti-color: #ff6ec7;
}

@keyframes fall {
  to {
    transform: translateY(110vh);
    opacity: 0.7;
  }
}

@keyframes sway {
  0%,
  100% {
    transform: translateX(0);
  }

  50% {
    transform: translateX(20px);
  }
}

.confetti {
  left: calc(100% * var(--random-x));
  animation-duration: calc(3s + 5s * var(--random-speed));
  animation-delay: calc(-10s * var(--random-delay));
}

.confetti:nth-child(1) {
  --random-x: 0.12;
  --random-speed: 0.9;
  --random-delay: 0.3;
}

.confetti:nth-child(2) {
  --random-x: 0.24;
  --random-speed: 0.5;
  --random-delay: 0.5;
}

.confetti:nth-child(3) {
  --random-x: 0.35;
  --random-speed: 0.7;
  --random-delay: 0.7;
}

.confetti:nth-child(4) {
  --random-x: 0.47;
  --random-speed: 0.4;
  --random-delay: 0.4;
}

.confetti:nth-child(5) {
  --random-x: 0.55;
  --random-speed: 0.6;
  --random-delay: 0.9;
}

.confetti:nth-child(6) {
  --random-x: 0.65;
  --random-speed: 0.8;
  --random-delay: 0.1;
}

.confetti:nth-child(7) {
  --random-x: 0.77;
  --random-speed: 0.3;
  --random-delay: 0.8;
}

.confetti:nth-child(8) {
  --random-x: 0.89;
  --random-speed: 0.6;
  --random-delay: 0.6;
}
</style>
