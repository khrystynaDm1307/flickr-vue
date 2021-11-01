<template>
  <div class="app">
    <div class="app-header">
      <h2>Flickr Search</h2>
      <div className="h-flex jc ac search-bar">
        <input
          type="text"
          className="search-input"
          v-model="searchText"
          v-on:input="changeSearchText()"
        />
      </div>

      <div v-if="commonTags.length">
        <h5>Suggestions</h5>
        <ul class="h-flex jc">
          <li
            v-for="(tag, index) in commonTags.slice(0, 5)"
            class="query"
            :key="index"
            v-on:click="changeSearchText(tag._content)"
          >
            {{ tag._content }}
          </li>
        </ul>
      </div>
    </div>

    <Loader v-if="isLoading" />

    <div className="app-content" ref="appContent" v-if="!isLoading">
      <ImageList v-bind:images="imageList" v-if="imageList.length" />
      <p v-if="!imageList.length">
        Try searching for some image in the search bar
      </p>
    </div>
  </div>
</template>

<script>
import constants from "./constants";
import { checkHttpStatus, parseJSON } from "./utils";
import Loader from "./components/Loader.vue";
import ImageList from "./components/ImageList.vue";

export default {
  name: "App",
  components: { Loader, ImageList },
  data() {
    return {
      searchText: "",
      imageList: [],
      commonTags: [],
      isLoading: false,
    };
  },
  methods: {
    changeSearchText() {
      const trimmedText = this.searchText.trim();
      if (trimmedText.length) {
        this.makeDebouncedSearch();
        this.getTags();
      }
    },
    makeDebouncedSearch() {
      /* Make API call for the query */
      const url = constants.BASE_URL + "&tags=" + this.searchText;
      this.isLoading = true;
      fetch(url)
        .then(checkHttpStatus)
        .then(parseJSON)
        .then((resp) => {
          this.isLoading = false;
          this.imageList = resp.photos.photo;
        })
        .catch((err) => {
          this.isLoading = false;
          console.log(err);
        });
    },

    getTags() {
      /* Make API call for the query */
      const url = constants.TAGS_URL + this.searchText.trim();

      fetch(url)
        .then(checkHttpStatus)
        .then(parseJSON)
        .then((resp) => {
          this.commonTags = resp.tags.tag;
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>

<style>
/* Common class styles used across all app */
.h-flex {
  display: flex;
  flex-wrap: wrap;
}
h2 {
  margin: 1rem 0;
}
.h-flex.jc {
  justify-content: center;
}
.h-flex.ac {
  align-items: center;
}
.h-flex.sb {
  justify-content: space-between;
}

.v-flex {
  display: flex;
  flex-wrap: wrap;
}
.v-flex.ac {
  align-items: center;
}
/**************************************/

/* App styles */
.app {
  text-align: center;
}

.app-header {
  background-color: #222;
  padding: 20px;
  color: white;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  margin: auto;
  z-index: 2;
  min-height: 150px;
}

.search-input {
  width: 50%;
  margin: 0 20px;
  border-radius: 2px;
  border: none;
  padding: 6px;
}

.app-content {
  position: relative;
  margin-top: 190px;
  padding: 30px;
}
.grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr 2fr;
  grid-auto-rows: 300px;
  grid-gap: 20px;
}
img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.query {
  font-size: 10px;
  background: grey;
  padding: 4px;
  border-radius: 2px;
  margin: 0 5px;
}
.btn {
  margin-left: 10px;
}

.image-item {
  cursor: pointer;
  animation: all 0.3s ease-in-out;
  margin: 8px;
}
li {
  list-style: none;
}

.image-metadata {
  font-size: 13px;
  transition: all 0.3s ease-in-out;
  margin: 20px;
  color: #fff;
}
</style>
