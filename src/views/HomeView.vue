
<script setup lang="ts">
//import TheWelcome from '../components/TheWelcome.vue'
</script>

<template>
  <div>
    <div class="flex">
      <input v-model="bvNumber" type="text" class="my-5 w-4/5 px-4 py-2 rounded-md bg-transparent ring-2 focus:rounded-lg focus:ring-2 focus:ring-blue-500 focus:outline-none" placeholder="视频BV号">
      <button @click="resolveVideo" class="w-1/5 my-5 ml-5 bg-blue-500 text-white px-4 rounded-md hover:bg-blue-600 focus:outline-none">
        解析
      </button>
    </div>

    <div class="my-2">
      <img :src="imageUrl" alt="Image" class="h-1/1 aspect-video rounded-md">
    </div>

    <div class="mt-8 shadow-md">
      <button @click="downloadVideo" :class="{ 'bg-blue-500 hover:bg-blue-600': isDownloadButtonEnabled, 'dark:bg-gray-500 bg-gray-400': !isDownloadButtonEnabled }" :disabled="!isDownloadButtonEnabled" class="w-full text-white px-4 py-2 rounded-md focus:outline-none">
        下载
      </button>
    </div>

    <div class="flex">
      <!-- 下载进度条 -->
      <div class="progress-bar-container w-11/12 h-2  dark:bg-gray-500 bg-gray-300 mt-4 rounded-md">
        <div class="progress-bar h-full bg-blue-500 rounded-md" :style="{ width: downloadProgress + '%' }"></div>
      </div>
      <!-- 下载速度 -->
      <div class="download-speed ml-2 mt-2 " v-if="downloadSpeed !== null">
        {{ downloadSpeed.toFixed(1) }}MB/s
      </div>
    </div>

  </div>
</template>


<script lang="ts">
import { ref } from 'vue';
import axios from 'axios';

const defaultImageUrl = 'https://i0.hdslb.com/bfs/article/32d48f7821243b471e2be2ebbbda76a53546375274892142.jpg@1e_1c.webp';

const bvNumber = ref('');
const imageUrl = ref(defaultImageUrl);
const isDownloadButtonEnabled = ref(false);
const downloadProgress = ref(0);
const downloadSpeed = ref(0);
const updateFrequency = 5; // 更新频率(ms)
let updateCounter = 0;

const resolveVideo = async () => {
  try {
    if (!bvNumber.value || !bvNumber.value.startsWith('BV')) {
      imageUrl.value = defaultImageUrl;
      isDownloadButtonEnabled.value = false;
      alert('请正确输入bv号');
      return;
    }

    const response = await axios.get(`https://bili-proxy.sccube.link/x/web-interface/view?bvid=${bvNumber.value}`);
    const picUrl = response.data.data.pic;
    imageUrl.value = picUrl;
    isDownloadButtonEnabled.value = true;

  } catch (error) {
    console.error('Error fetching data:', error);
    imageUrl.value = defaultImageUrl;
    isDownloadButtonEnabled.value = false;
    alert('解析失败');
  }
};

/*--download--*/
const downloadVideo = () => {
  if (!isDownloadButtonEnabled.value) {
    return;
  }
  if (!bvNumber.value || !bvNumber.value.startsWith('BV')) {
    alert('请正确输入bv号');
    return;
  }

  //window.open会在新标签页打开视频
  /*
  const downloadUrl = `https://bili.api.scc.lol/?bv=${bvNumber.value}`;
  window.open(downloadUrl, '_blank');
  */

  const downloadUrl = `https://bili.api.scc.lol/?bv=${bvNumber.value}`;

  // 计算下载速度
  let prevTime = performance.now();
  let prevLoaded = 0;

  //加载
  const xhr = new XMLHttpRequest();
  xhr.open('GET', downloadUrl, true);
  xhr.responseType = 'blob';

  // progress 监听进度
  xhr.addEventListener('progress', (event) => {
    if (event.lengthComputable) {
      const currentTime = performance.now();
      const elapsedTime = (currentTime - prevTime) / 1000; // s
      const loadedData = event.loaded - prevLoaded;
      const speed = loadedData / elapsedTime / 1024 / 1024; // MB/s
      
      const percentComplete = (event.loaded / event.total) * 100;
      downloadProgress.value = percentComplete;

      if (updateCounter === 0) {
        downloadSpeed.value = speed;
      }
      updateCounter = (updateCounter + 1) % updateFrequency;
      prevTime = currentTime;
      prevLoaded = event.loaded;
    }
  });

  xhr.onload = function () {
    const blob = new Blob([xhr.response], { type: 'video/mp4' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'video.mp4';
    document.body.appendChild(a);
    a.click();

    URL.revokeObjectURL(url);
    document.body.removeChild(a);

    // 重置
    downloadProgress.value = 0;
    downloadSpeed.value = 0;
  };

  xhr.send();

};
</script>

