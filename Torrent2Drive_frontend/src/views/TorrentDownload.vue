<template>
    <div>
        <button class="btn btn-info" @click="showDownloads">My current Downloads</button>
        <div class="download-form">
            <form @submit.prevent="onSubmit">
                <input
                      type="checkbox"
                      id="zip"
                      value="true"
                      v-model="doZip"
                >
                <label for="zip"> Zip all torrent files?</label>
                <div class="input" :class="{invalid: $v.magnet.$error}">
                    <label for="magnet">Magnet</label>
                    <input
                            type="text"
                            id="magnet"
                            placeholder="magnet link here"
                            v-model="magnet"
                            @blur="$v.magnet.$touch()"
                    >
                </div>
                <p v-if="$v.magnet.$error" class="alert alert-danger">Please enter a valid magnet</p>
                <p v-if="error" class="alert alert-danger">{{error}}</p>
                <div class="submit">
                    <button @click="showStats=!showStats" type="submit" :disabled="!$v.magnet.containsMagnet" ref="downloadButton">Download</button>
                </div>
            </form>
        </div>
        <app-download-status v-if="showStats" :downloadCompleted="downloadCompleted"></app-download-status>
        <app-download-complete v-if="finished" :doZip="doZip"></app-download-complete>
    </div>
</template>

<script>
import { required } from 'vuelidate/lib/validators'
import axios from 'axios'
import { mapGetters } from 'vuex'
import DownloadStats from '../components/DownloadStatus'
import DownloadComplete from '../components/DownloadComplete'
export default {
  data () {
    return {
      magnet: '',
      finished: false,
      showStats: false,
      error: null,
      doZip: false
    }
  },
  validations: {
    magnet: {
      required,
      containsMagnet: magnet => {
        return magnet.includes('magnet') || magnet.includes('torrent')
      }
    }
  },
  computed: {
    ...mapGetters({
      user: 'getUser'
    })
  },
  components: {
    appDownloadStatus: DownloadStats,
    appDownloadComplete: DownloadComplete
  },
  methods: {
    onSubmit () {
      this.$refs.downloadButton.innerText = 'Downloading'
      this.$store.state.stats = null
      this.finished = false
      axios.post('/download', { magnet: this.magnet, doZip: this.doZip })
        .then(res => {
          console.log(res.data)
          this.finished = res.data.finished
          this.showStats = false
          this.$refs.downloadButton.innerText = 'Download'
        })
        .catch(err => {
          if (err.statusCode === 500) {
            this.error = 'Invalid magnet link detected!! Enter a valid magnet.'
            setTimeout(() => {
              this.error = null
            }, 3000)
          }
        })
    },
    showDownloads () {
      if (this.$store.getters.getStats) {
        this.showStats = true
      }
    },
    downloadCompleted () {
      this.magnet = ''
      this.showStats = false
      this.finished = true
      this.$refs.downloadButton.innerText = 'Download'
      this.doZip = false
    }
  },
  created () {
    this.$store.dispatch('setUser')
    this.$store.dispatch('setStats')
  }
}
</script>

<style scoped>
    .download-form {
        width: 600px;
        margin: 30px auto;
        border: 1px solid #eee;
        padding: 20px;
        box-shadow: 0 19px 38px rgba(0,0,0,0.30), 0 15px 12px rgba(0,0,0,0.22);
    }
    .input {
        margin: 10px auto;
    }

    .input label {
        display: block;
        color: black;
        margin-bottom: 6px;
    }

    .input.inline label {
        display: inline;
    }

    .input input {
        font: inherit;
        width: 100%;
        padding: 6px 12px;
        box-sizing: border-box;
        border: 1px solid #ccc;
    }

    .input.inline input {
        width: auto;
    }

    .input input:focus {
        outline: none;
        border: 1px solid #521751;
        background-color: #eee;
    }
    .input.invalid input {
        border: 1px solid red;
        background-color: #ffc9aa;
    }
    .input.invalid label {
        color: red;
    }
    .submit button {
        border: 1px solid #521751;
        color: #007bff;
        padding: 10px 20px;
        font: inherit;
        cursor: pointer;
        display: inline;
    }
    .submit button:hover,
    .submit button:active {
        background-color: #007bff;
        color: white;
    }
    .submit button[disabled],
    .submit button[disabled]:hover,
    .submit button[disabled]:active {
        border: 1px solid #ccc;
        background-color: transparent;
        color: #ccc;
        cursor: not-allowed;
    }

    input[type=checkbox] + label {
      display: block;
      margin: 0.2em;
      cursor: pointer;
      padding: 0.2em;
    }

    input[type=checkbox] {
      display: none;
    }

    input[type=checkbox] + label:before {
      content: "\2714";
      border: 0.1em solid #000;
      border-radius: 0.2em;
      display: inline-block;
      width: 1.5em;
      height: 1.5em;
      padding-left: 0.2em;
      padding-bottom: 0.3em;
      margin-right: 0.2em;
      vertical-align: bottom;
      color: transparent;
      transition: .2s;
    }

    input[type=checkbox] + label:active:before {
      transform: scale(0);
    }

    input[type=checkbox]:checked + label:before {
      /* background-color: MediumSeaGreen; */
      border-color: MediumSeaGreen;
      color: #fff;
    }

    input[type=checkbox]:disabled + label:before {
      transform: scale(1);
      border-color: #aaa;
    }

    input[type=checkbox]:checked:disabled + label:before {
      transform: scale(1);
      background-color: #bfb;
      border-color: #bfb;
    }
</style>