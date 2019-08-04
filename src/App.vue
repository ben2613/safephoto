<template>
  <div id="app">
    <input type="file" @change="onFileChange" accept="image/*;capture=camera" />

    <h3>Click to get a safe-to-share image</h3>
    <div id="preview">
      <img :src="url" v-if="url" @click="clearAndSave"/>
    </div>

    <div id="exif-details" v-if="exif">
      <div v-for="(tagGroup, name) in exifTags" v-bind:key="name">
        <div v-for="(tag, i) in tagGroup" v-bind:key="i">
          {{tag[0]}}: {{exif[name][tag[1]]}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const piexif = require("piexifjs");
const FileSaver = require('file-saver');
window.piexif = piexif;

export default {
  name: "app",
  data() {
    return {
      readResult: null,
      url: null,
      exif: null,
      exifTags: {
        "0th": [
          ["Make", piexif.ImageIFD.Make],
          ["Model", piexif.ImageIFD.Model],
          ["Software", piexif.ImageIFD.Software],
          ],
        "Exif": [
          ["DateTimeOriginal", piexif.ExifIFD.DateTimeOriginal]
        ],
        "GPS": [
          ["GPSLatRef", piexif.GPSIFD.GPSLatitudeRef],
          ["GPSLat", piexif.GPSIFD.GPSLatitude],
          ["GPSLongRef", piexif.GPSIFD.GPSLongitudeRef],
          ["GPSLong", piexif.GPSIFD.GPSLongitude],
        ],
        "Interop": [],
        "1st": []
      }
    };
  },
  methods: {
    onFileChange(e) {
      var file = e.target.files[0];
      const reader = new FileReader();
      reader.onloadend = this.onReaderLoadEnd;
      reader.readAsDataURL(file);
    },
    onReaderLoadEnd(e) {
      this.readResult = e.target.result
      this.exif = piexif.load(this.readResult);
      var jpg = piexif.remove(this.readResult);
      this.url = jpg
    },
    clearAndSave(e) {
      FileSaver.saveAs(this.url, 'safeimage.jpg')
    }
  }
};
</script>

<style scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#preview img {
  max-width: 100vw;
  max-height: 90vh;
}
</style>
