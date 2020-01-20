<template>
  <div>
    <!-- <div
      class="area_1"
    >
      <div class="v-bar-holder">
        <div class="content-list">
          <section class="archived-section mb-2">
            <h4 class="common-heading">
              <div class="row">
                <div class="col">
                  MarketPlace Applications
                </div>
              </div>
            </h4>
            <div class="row offset-2">
              <div class="row">
                <form>
                  <h2>Upload File</h2>
                  Enter FileName to  upload : <input
                    id="files"
                    type="text"
                    v-model="inputFileName"
                  ><br>
                  <input
                    id="files"
                    @change="submitFile"
                    type="file"
                  >
                  <br>
                  <br>
                  <button
                    id="uploadtBtn"
                    @click.stop.prevent="uploadFile"
                  >
                    Upload
                  </button>
                  <span v-if="uploadedFilePath!==''">
                    Uploaded FilePath is : {{ uploadedFilePath }}
                  </span>
                  <h2>Download File</h2>
                  Enter FileURl to download : <input
                    id="files"
                    type="text"
                    v-model="fileURL"
                  ><br>
                  <br>
                  <button
                    id="downloadBtn"
                    @click.stop.prevent="downloadFile"
                  >
                    Download
                  </button>
                  <br>
                  <br>
                </form>
              </div>
            </div>
            <div class="row offset-2">
              Show Image
              <img
                width="200px"
                height="150px"
                :src="cdnBaserURl+'client2/loginIP.png'"
              >
            </div>
          </section>
        </div>
      </div>
    </div> -->
    <div class="container">
      <div class="row text-left mt-3">
        <div class="col-sm-6">
          <div class="card p-3">
            <div class="form-group">
              <h5>Upload File</h5>
              <input
                class="form-control mb-2"
                id="files"
                @change="submitFile"
                type="file"
              >
              <button
                type="button"
                class="btn btn-primary btn-sm"
                id="uploadtBtn"
                @click.stop.prevent="uploadFile2"
              >
                Upload
              </button>
              <input
                class="form-control mb-2"
                placeholder="CDN file path"
                v-model="uploadedFilePath"
              >
            </div>
          </div>
        </div>
        <div class="container">
          <div class="row text-left mt-3">
            <div class="col-sm-6">
              <div class="card p-3">
                <div class="form-group">
                  <h5>Upload File</h5>
                  <input
                    class="form-control mb-2"
                    id="files"
                    @change="submitFile"
                    type="file"
                  >
                  <button
                    type="button"
                    class="btn btn-primary btn-sm"
                    id="uploadtBtn"
                    @click.stop.prevent="uploadFile"
                  >
                    Upload
                  </button>
                </div>
              </div>
            </div>
            <!-- <div class="col-sm-6">
          <div class="card p-3">
            <div class="form-group">
              <h5>Download File</h5>
              <label for=""> Enter FileURl to download</label>
              <input
                id="files"
                type="text"
                v-model="fileURL"
                class="form-control mb-2"
              >
              <button
                id="downloadBtn"
                class="btn btn-primary btn-sm"
                @click.stop.prevent="downloadFile"
              >
                Download
              </button>
            </div>
            <div class="form-group">
              <label
                class="d-block"
                for=""
              > Show Image</label>
              <img
                width="200px"
                height="150px"
                :src="cdnBaserURl+'client2/loginIP.png'"
              >
            </div>
          </div>
        </div>
      </div>
    </div> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MQL from '@/plugins/mql.js'
import MQLCdn from '@/plugins/mqlCdn.js'
import Vue from 'vue'
import XLSX from 'xlsx'
// import vueDropzone from 'vue2-dropzone'
// import axios from 'axios'
export default {
  // components: {
  //   vueDropzone
  // },
  data () {
    return {
      fileURL: '',
      files: '',
      uploadedFilePath: '',
      cdnBaserURl: Vue.getCDNBaseURL()
    }
  },
  methods: {
    submitFile (event) {
      this.files = event.target.files[0]
    },
    uploadFile2 () {
      let formData = new FormData()
      formData.append('file', this.files) // append your file as 'file' in formdata.
      new MQLCdn()
        .enablePageLoader(true)
        // FIXED: change this to directory path
        .setDirectoryPath('/demoFolder') // (optional field) if you want to save  file to specific directory path
        .setFormData(formData) // (required) sets file data
        .setFileName(this.inputFileName) // (optional field) if you want to set name to file that is being uploaded
        // FIXED: pass buckeyKey instead of name
        .setBucketKey('1TxYD2KhMcczFlxXntsueOYN46J') // (required) valid bucket key need to set in which file will be uploaded.
        .setPurposeId('1TxY9TS4uzp8Ivyo0eKPpo1g2Og') // (required) valid purposeId need to set in which file will be uploaded.
        .setClientId('1TxY9TS4uzp8Ivyo0eKPpo1g2Og') // (required) valid purposeId need to set in which file will be uploaded.
        .uploadFile('uploadtBtn').then(res => { // (required) this will upload file takes element id (optional param) which will be blocked while file upload..
          if (res.isValid()) {
            this.uploadedFilePath = res.uploadedFileURL().cdnServer + '/' + res.uploadedFileURL().filePath // returns uploaded file url..
            console.log('res cdn path', this.uploadedFilePath)
            new MQL()
              .setActivity('o.[Excel]')
              .setData({ fileData: this.uploadedFilePath })
              .fetch()
              .then(rs => {
                let res = rs.getActivity('Excel')
                if (rs.isValid('Excel')) {
                  // Write some intelligent logic here
                  console.log('Data Received:  ', res.result)
                } else {
                  rs.showErrorToast('Excel')
                }
              })
            this.$toasted.success('file uploaded.', {
              theme: 'bubble',
              position: 'top-center',
              duration: 5000
            })
          } else {
            res.showErrorToast()
          }
        })
    },
    uploadFile () {
      if (this.files === null) {
        this.$toasted.error('Please add file')
        return
      }
      var workbook
      var reader = new FileReader()
      let self = this
      reader.onload = function (e) {
        let data = e.target.result
        let fixedData = self.fixData(data)
        workbook = XLSX.read(btoa(fixedData), { type: 'base64' })
        if (workbook.Sheets.length === 0) {
          self.$toasted.error('Empty Excel With No sheet')
        }
        let obj = []
        let rows = XLSX.utils.sheet_to_json(
          workbook.Sheets[workbook.SheetNames[0]],
          { header: ['name', 'designation', 'emp_id'] }
        )
        if (rows.length <= 1) {
          self.$toasted.error('Excel Parsed with no data')
        }
        for (let i = 1; i < rows.length; i++) {
          let row = rows[i]
          obj.push(row)
        }
        if (obj.length === 0) {
          self.$toasted.error('Users already added ... please resend invitation')
        }

        new MQL()
          .setActivity('o.[Excel]')
          .setData(obj)
          .fetch()
          .then(rs => {
            let res = rs.getActivity('Excel')
            if (rs.isValid('Excel')) {
            // Write some intelligent logic here
              console.log('Data Received:  ', res.result)
            } else {
              rs.showErrorToast('Excel')
            }
          })
      }
      reader.readAsArrayBuffer(this.files)
    },
    fixData (data) {
      var o = ''
      var l = 0
      var w = 10240
      for (; l < data.byteLength / w; ++l) {
        o += String.fromCharCode.apply(
          null,
          new Uint8Array(data.slice(l * w, l * w + w))
        )
      }
      o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
      return o
    }
  }
}
</script>

<style>

</style>
