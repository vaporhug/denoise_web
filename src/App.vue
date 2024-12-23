<script setup>
import axios from 'axios';
import ImageCompare from './components/ImageCompare.vue';
import originalImg from './assets/original_img.png';
import denoisedImg from './assets/denosied_img.png';

const originalImage = denoisedImg;
const denoisedImage = originalImg;

const handleImageUpload = async (event) => {
  const file = event.target.files[0];
  if (!file) return;

  const formData = new FormData();
  formData.append('image', file);

  try {
    const response = await axios.post('/api/denoise', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });

    const denoisedImageUrl = response.data.denoised_image_url;
    const originalImageUrl = URL.createObjectURL(file);

    this.originalImage = originalImageUrl;
    this.denoisedImage = denoisedImageUrl;
  } catch (error) {
    console.error('图像上传或去噪处理失败:', error);
    alert('图像上传或去噪处理失败，请重试。');
  }
};

const handleImageDownload = () => {
  const link = document.createElement('a');
  link.href = denoisedImage;
  link.download = 'denoised_image.png';
  link.click();
};
</script>

<template>
  <div id="app">
    <h1>图像去噪网站</h1>
    <div class="image-display">
      <ImageCompare :originalImage="originalImage" :denoisedImage="denoisedImage" />
    </div>
    <div class="button-section">
      <label for="image-upload" class="upload-button">上传图片</label>
      <input
          id="image-upload"
          type="file"
          @change="handleImageUpload"
          accept="image/*"
          class="file-input"
      />
      <button @click="handleImageDownload" class="download-button">下载图片</button>
    </div>
  </div>
  <!-- Footer -->
  <footer class="footer">
    <p>&copy; 2024 图像降噪网站. 保留所有权利。</p>
  </footer>
</template>

<style scoped>
#app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 90vh;
  width: 100%;
  background-color: #f4f6f9;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  font-size: 36px;
  background: linear-gradient(135deg, #a1c4fd, #84fab0); /* 渐变色：蓝到绿 */
  -webkit-background-clip: text;
  color: transparent;
  margin-bottom: 30px;
}

.image-display {
  margin-bottom: 30px;
  border-radius: 15px; /* 圆角 */
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1); /* 阴影效果 */
  overflow: hidden; /* 确保图片不会超出圆角边界 */
}

.button-section {
  display: flex;
  gap: 20px;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.upload-button,
.download-button {
  padding: 12px 30px;
  font-size: 16px;
  border-radius: 8px;
  cursor: pointer;
  border: none; /* 去除边框 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 阴影效果 */
}

.upload-button {
  background: linear-gradient(135deg, #a1c4fd, #c2e9fb); /* 淡蓝渐变 */
  color: white;
  transition: background-color 0.3s ease;
}

.upload-button:hover {
  background: linear-gradient(135deg, #6a99d1, #9cd1e7); /* 悬停渐变 */
}

.download-button {
  background: linear-gradient(135deg, #b3f7a7, #d6ffcc); /* 淡绿渐变 */
  color: white;
  transition: background-color 0.3s ease;
}

.download-button:hover {
  background: linear-gradient(135deg, #88e08d, #b4f1b1); /* 悬停渐变 */
}

.file-input {
  display: none;
}

input[type="file"]:focus {
  outline: none;
}

input[type="file"]::-webkit-file-upload-button {
  display: none;
}

/* Footer */
.footer {
  width: 100%;
  background-color: #ffffff; /* 深色背景 */
  color: #7be0d5;
  text-align: center;
  padding: 10px 0;
  margin-top: 20px;
}

</style>
