<template>
  <div class="container">
    <header class="header">Record and Watch</header>
    <div class="alert">{{ myError }}</div>

    <div class="video-container">
      <video class="video" ref="videoBox" autoplay></video>
    </div>

    <div class="button-container">
      <button
        class="action-button"
        @click="startRecording"
        :disabled="isRecording"
      >
        Start Recording
      </button>
      <button
        class="action-button"
        @click="stopRecording"
        :disabled="!isRecording"
      >
        Stop Recording
      </button>
    </div>

    <div v-if="recordedVideoLink" class="recorded-video-container">
      <video :src="recordedVideoLink" controls class="recorded-video"></video>
      <button @click="reRecord" class="action-button">Re-record</button>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";
export default {
  setup() {
    const isRecording = ref(false);
    const recordedVideo = ref<Blob | null>(null);
    const recordedVideoLink = ref<string | null>(null);
    let myError = ref<string | null>(null);
    let mediaStream: MediaStream | null = null;
    let mediaRecorder: MediaRecorder | null = null;

    const startRecording = async () => {
      if (!isRecording.value) {
        try {
          const stream = await navigator.mediaDevices.getUserMedia({
            video: true,
          });
          mediaStream = stream;
          const videoBox: HTMLVideoElement | null =
            document.querySelector("video");
          if (videoBox) {
            videoBox.srcObject = stream;
          }

          mediaRecorder = new MediaRecorder(stream);

          mediaRecorder.ondataavailable = (event) => {
            if (event.data.size > 0) {
              recordedVideo.value = new Blob([event.data], {
                type: "video/webm",
              });
              recordedVideoLink.value = URL.createObjectURL(
                recordedVideo.value
              );
            }
          };

          mediaRecorder.start();
          isRecording.value = true;
        } catch (err: any) {
          myError.value = err;
        }
      }
    };

    const stopRecording = () => {
      if (mediaStream && isRecording.value) {
        isRecording.value = false;

        if (mediaRecorder) {
          mediaRecorder.stop();
        }

        mediaStream.getTracks().forEach((track) => track.stop());
      }
    };

    const reRecord = () => {
      if (recordedVideo.value) {
        recordedVideo.value = null;
        recordedVideoLink.value = null;
      }
    };

    return {
      isRecording,
      startRecording,
      stopRecording,
      recordedVideoLink,
      reRecord,
      myError,
    };
  },
};
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  background: #f4f4f4;
  padding: 20px;
  text-align: center;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
  max-width: 800px;
  margin: 0 auto;
}

.header {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #333;
}

.alert {
  color: red;
  font-weight: bold;
  margin-bottom: 10px;
}

.video-container {
  background: #333;
  width: 100%;
  margin: auto;
  text-align: center;
  border-radius: 10px;
  overflow: hidden;
}

.video {
  width: 100%;
  height: auto;
  border-radius: 10px;
  max-height: 400px;
}

.button-container {
  display: flex;
  justify-content: center;
}

.action-button {
  margin: 5px;
  background: #ff7f50;
  color: #fff;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  transition: background 0.3s;
}

.action-button:hover {
  background: #ff6347;
}

.recorded-video-container {
  margin-top: 20px;
}

.recorded-video {
  width: 100%;
  height: auto;
  border-radius: 10px;
  max-height: 400px;
}

@media (max-width: 768px) {
  .container {
    max-width: 100%;
    border-radius: 0;
  }
  .video-container {
    border-radius: 0;
  }
}
</style>
