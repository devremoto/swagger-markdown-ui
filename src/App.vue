<template>
  <div class="mt-2">
    <h1 class="mb-4" id="top">Swagger Markdown Preview</h1>

    <div class="mb-3">
      <label for="urlInput" class="form-label">Swagger JSON URL</label>
      <input v-model="swaggerUrl" @change="reset" type="text" class="form-control" id="urlInput"
        placeholder="Enter Swagger JSON URL">
    </div>

    <div class="mb-3">
      <label for="baseUrlInput" class="form-label">Base URL</label>
      <input v-model="baseUrl" type="text" class="form-control" id="baseUrlInput" placeholder="Enter Base URL">
    </div>

    <div class="mb-3">
      <label for="baseUrlInput" class="form-label">Swagger</label>
      <textarea class="form-control form-input" rows="5" v-model="content" @change="reset" ></textarea>
    </div>

    <button @click="generatePreview()" :disabled="!(swaggerUrl || content)" class="btn btn-primary">Generate Preview</button>

    <hr class="my-4">

    <div v-if="readme" class="row">
      <div class="col-4">
        <label class="form-label">Readme</label>
        <textarea class="form-control form-input" rows="5" v-html="readme"></textarea>
      </div>
      <div class="col-4 px-2">
        <label class="form-label">Source</label>
        <textarea class="form-control" rows="5" v-html="source"></textarea>
      </div>
      <div class="col-4">
        <label class="form-label">Json</label>
        <textarea class="form-control" rows="5" v-html="json"></textarea>
      </div>
      <label class="form-label mt-5">Preview</label>
      <div v-html="showDown()" class="col-12 card md-container inner-shadow"></div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { marked } from 'marked';
import { ref } from 'vue';
export default {
  data() {
    return {
      swaggerUrl: '',
      baseUrl: '',
      content: '',
      readme: '',
      source: '',
      json: {}
    };
  },
  computed: {

  },
  methods: {
    showDown() {
      this.reset();
      if (!(this.swaggerUrl || this.content)) {
        return;
      }
      return this.anchor(marked(this.readme));
    },
    anchor(htmlString) {
      var headerRegex = /<h([1-6])>(.*?)<\/h\1>/gi;

      var replacedHtml = htmlString.replace(headerRegex, function (match, tagNumber, innerHtml) {
        var id = innerHtml.toLowerCase().replace(/\s/g, '-');
        return `<h${tagNumber} id="${id}"><a class="link-none" href="#top" title="back to top">${innerHtml}</a></h${tagNumber}>`;
      });

      return replacedHtml;
    },
    reset() {
      if (!(this.swaggerUrl || this.content)) {
        this.readme = ''
        this.source = ''
        this.json = {}
      }
    },
    async generatePreview() {
      try {
        this.reset();
        if (!(this.swaggerUrl || this.content)) {
          return;
        }
        let url = `http://localhost:3000/preview`
        const response = await axios.post(url, this)
        let { readme, source, sourceJson: json } = response.data
        console.log(response);
        this.readme = readme;
        this.source = source;
        this.json = json;
        console.log({ readme, source, json })
      } catch (error) {
        console.error('Error fetching preview:', error);
      }
    }
  }
}
</script>

<style>
.preview-container {
  text-align: left;
}

.form-label {
  font-weight: bold
}

table,
td,
th {
  border: #efefef solid 1px;
  margin-bottom: 40px;
}

table {
  width: 100%;
}

th,
td {
  padding: 5px;
  width: auto;
}

.inner-shadow {
  -webkit-box-shadow: inset 2px 2px 2px 0px #dddddd;
  -moz-box-shadow: inset 2px 2px 2px 0px #dddddd;
  box-shadow: inset 2px 2px 2px 0px #dddddd;

}

.md-container {
  max-height: 300px;
  overflow: auto;
}
</style>
