<template>
  <ul className="grid">
    <div v-for="(image, idx) in images" :key="idx">
      <li class="image-item">
        <p>{{ renderData(image, idx) }}</p>
        <img :src="renderImageItem(image)" alt="" />
      </li>
    </div>
  </ul>
</template>

<script>
import constants from "../constants";
import {
  checkHttpStatus,
  parseJSON,
  getImageUrl,
  getIntervalOfTime,
} from "../utils";

export default {
  name: "ImageList",
  props: ["images"],
  data() {
    return {
      dates: []
    };
  },
  methods: {
    renderImageItem(image) {
      const { farm, server, id, secret } = image;
      this.getPhotoInfo(id, secret);
      const url = getImageUrl(farm, server, id, secret);

      return url;
    },
    renderData(image,idx) {
      const {  id, secret } = image;
      this.getPhotoInfo(id, secret);
      const data = getIntervalOfTime(this.dates[idx]);

      return data;
    },

    getPhotoInfo(id, secret) {
      const url = constants.DATE_URL + id + "&secret=" + secret;
      fetch(url)
        .then(checkHttpStatus)
        .then(parseJSON)
        .then((resp) => {
          this.dates.push(resp.photo.dates.taken);
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr 2fr;
  grid-auto-rows: 300px;
  grid-gap: 20px;
}
img{
    width: 300px;
}
</style>
