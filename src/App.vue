<template>
  <div class="row mt-2">
    <h1 class="mb-4" id="top">Swagger Markdown Preview</h1>

    <div class="mb-3">
      <label for="urlInput" class="form-label">Swagger JSON URL</label>
      <input v-model="url" @change="reset" type="text" class="form-control" id="urlInput"
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
    <div class="mb-3">
    <button @click="generatePreview()" :disabled="!(url || content)" class="btn btn-primary">Generate Preview</button>
    <button @click="clear()" :disabled="!readme" class="btn btn-warning m-2">Reset</button></div>
</div>

    <div v-if="readme" class="row mt-2"> 
      <div class="col-8">
        <label class="form-label">Preview</label> 
      </div>
      <div class="col-4 text-end">
          <span id="copyBtn" class="me-3" role="button" title="Copy to clipboard"

            @click="copyToClipboard(readme)">
            <i class="bi bi-clipboard"></i>
          </span>
          <span id="downloadBtn" @click="download(readme)" role="button" title="Download Markdown file">           
            <i class="bi bi-download"></i></span>
        </div>
      <div class="col-12 preview-container mt-2 mb-5">
         <div v-html="showDown()" class="card md-container inner-shadow"></div>
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
      url: '',
      baseUrl: '',
      content: '',
      readme: '',
      source: '',
      json: {},
      fileName: ''
    };
  },
  computed: {

  },
  methods: {
    showDown() {
      this.reset();
      if (!(this.url || this.content)) {
        return;
      }
      return this.anchor(marked(this.readme));
    },
    anchor(htmlString) {
      var headerRegex = /<h([1-6])>(.*?)<\/h\1>/gi;

      var replacedHtml = htmlString.replace(headerRegex, function (match, tagNumber, innerHtml) {
        var id = innerHtml.toLowerCase().replace(/\s/g, '-');
        return `<h${tagNumber} id="${id}"><a class="link-none" href="#api-documentation" title="back to top">${innerHtml}</a></h${tagNumber}>`;
      });

      return replacedHtml;
    },
    clear() {
      this.readme = ''
      this.source = ''
      this.json = {}
    },
    reset() {
      if (!(this.url || this.content)) {
        this.readme = ''
        this.source = ''
        this.json = {}
      }
    },
    async generatePreview() {
      try {
        this.reset();
        if (!(this.url || this.content)) {
          return;
        }
        let apiUrl = `http://localhost:3000/preview`
        var params = {
          url: this.url,
          baseUrl: this.baseUrl,
          content: this.content
        }
        const response = await axios.post(apiUrl, params)
        let { readme, source, sourceJson: json } = response.data
        console.log(response);
        this.readme = readme;
        this.source = source;
        this.json = json;
        var dateTimeGenerated = new Date().toISOString().replace(/[:.]/g, '-');
        this.fileName = `API-DOCUMENTATION-${dateTimeGenerated}.md`;
        console.log({ readme, source, json })
      } catch (error) {
        console.error('Error fetching preview:', error);
      }
    },
    download(text) {
      const blob = new Blob([text], { type: 'text/markdown' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = this.fileName || 'API-DOCUMENTATION.md';
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    },
    async copyToClipboard(text) {
      navigator.clipboard.writeText(text).then(() => {
        alert('Copied to clipboard!');
      }, (err) => {
        console.error('Could not copy text: ', err);
      });
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
