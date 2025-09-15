<template>
  <div class="container">
    <h1>遺物一覧ジェネレーター</h1>

    <input type="file" multiple accept="image/*" @change="handleFiles" />

    <label>
      列数:
      <input type="number" v-model.number="columns" min="1" />
    </label>

    <button @click="generateImage" :disabled="previews.length === 0">画像を合成して生成</button>

    <div class="preview">
      <div v-for="(img, i) in previews" :key="i">
        <img :src="img" />
      </div>
    </div>

    <div v-if="resultImage">
      <h2>一覧画像</h2>
      <img :src="resultImage" />
      <a :href="resultImage" download="relic-list.png">ダウンロード</a>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      previews: [],
      images: [],
      resultImage: null,
      columns: 3,
    };
  },
  methods: {
    handleFiles(e) {
      const files = e.target.files;
      this.previews = [];
      this.images = [];
      for (const file of files) {
        const reader = new FileReader();
        reader.onload = (event) => {
          const img = new Image();
          img.onload = () => {
            this.images.push(img);
          };
          img.src = event.target.result;
          this.previews.push(event.target.result);
        };
        reader.readAsDataURL(file);
      }
    },
    generateImage() {
      const crop = {
        x: 104,
        y: 245,
        width: 620,
        height: 750,
      };

      const rows = Math.ceil(this.images.length / this.columns);
      const canvas = document.createElement('canvas');
      canvas.width = crop.width * this.columns;
      canvas.height = crop.height * rows;

      const ctx = canvas.getContext('2d');

      this.images.forEach((img, i) => {
        const col = i % this.columns;
        const row = Math.floor(i / this.columns);
        ctx.drawImage(
          img,
          crop.x,
          crop.y,
          crop.width,
          crop.height,
          col * crop.width,
          row * crop.height,
          crop.width,
          crop.height
        );
      });

      this.resultImage = canvas.toDataURL();
    }
  }
};
</script>

<style>
.container {
  max-width: 800px;
  margin: auto;
  text-align: center;
  padding: 20px;
}
.preview {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 20px;
}
.preview img {
  max-width: 150px;
  margin: 5px;
  border: 1px solid #ccc;
}
</style>
