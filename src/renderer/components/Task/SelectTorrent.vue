<template>
  <div>
    <el-upload
      class="upload-torrent"
      drag
      action="/"
      :limit="1"
      :multiple="false"
      accept=".torrent"
      :on-change="handleChange"
      :on-exceed="handleExceed"
      :auto-upload="false"
      :show-file-list="false">
      <i class="upload-inbox-icon"><mo-icon name="inbox" width="24" height="24" /></i>
      <div class="el-upload__text">
        {{ $t('task.select-torrent') }}
        <div class="torrent-name" v-if="name">{{ name }}</div>
      </div>
    </el-upload>
    <div class="torrent-files" v-if="torrentFiles.length !== 0">
      <div class="torrent-files-top">
        <span v-on:click="selectAll">全选</span>
        &nbsp;/&nbsp;
        <span v-on:click="selectNone">全不选</span>
      </div>
      <ul>
        <li v-for="(item, i) in torrentFiles">
          <el-checkbox class="chk" v-model="item.selected"><div>{{ item.name }}</div><div>{{ item.size }}</div></el-checkbox>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import { mapState } from 'vuex'
  import parseTorrent from 'parse-torrent'
  import '@/components/Icons/inbox'
  import { getAsBase64, buildFileList } from '@shared/utils'

  export default {
    name: 'mo-select-torrent',
    components: {
    },
    props: {
    },
    data () {
      return {
        name: '',
        torrentFiles: []
      }
    },
    computed: {
      ...mapState('preference', {
        config: state => state.config
      }),
      ...mapState('app', {
        torrents: state => state.addTaskTorrents
      }),
      selectFileStr: function () {
        let str = ''
        let torrentFiles = this.torrentFiles
        for (let index = 0; index < torrentFiles.length; index++) {
          if (torrentFiles[index].selected) {
            str += (index + 1) + ','
          }
        }
        if (str.length > 0) {
          str = str.substr(0, str.length - 1)
        }
        return str
      }
    },
    watch: {
      torrents (fileList) {
        const file = fileList[0]
        if (fileList.length === 0) {
          this.name = ''
          this.torrentFiles = []
          return
        }
        if (!file.raw) {
          return
        }

        parseTorrent.remote(file.raw, (err, parsedTorrent) => {
          if (err) throw err
          console.log(parsedTorrent)
          for (let index in parsedTorrent.files) {
            parsedTorrent.files[index].selected = true
            if (parsedTorrent.files[index].length >= 1024 * 1024 * 1024) {
              parsedTorrent.files[index].size = (parsedTorrent.files[index].length / (1024 * 1024 * 1024)).toFixed(2) + 'GiB'
            } else if (parsedTorrent.files[index].length >= 1024 * 1024) {
              parsedTorrent.files[index].size = (parsedTorrent.files[index].length / (1024 * 1024)).toFixed(2) + 'MiB'
            } else if (parsedTorrent.files[index].length >= 1024) {
              parsedTorrent.files[index].size = (parsedTorrent.files[index].length / 1024).toFixed(2) + 'KiB'
            } else {
              parsedTorrent.files[index].size = parsedTorrent.files[index].length + 'B'
            }
          }
          this.torrentFiles = parsedTorrent.files
        })

        getAsBase64(file.raw, (torrent) => {
          this.name = file.name
          this.$emit('change', torrent, file, fileList, '')
        })
      },
      selectFileStr (val) {
        const file = this.torrents[0]
        getAsBase64(file.raw, (torrent) => {
          this.name = file.name
          this.$emit('change', torrent, file, this.torrents, val)
        })
      }
    },
    methods: {
      handleChange (file, fileList) {
        this.$store.dispatch('app/addTaskAddTorrents', { fileList })
      },
      handleExceed (files) {
        const fileList = buildFileList(files[0])
        this.$store.dispatch('app/addTaskAddTorrents', { fileList })
      },
      selectAll: function () {
        for (let index in this.torrentFiles) {
          this.torrentFiles[index].selected = true
        }
      },
      selectNone: function () {
        for (let index in this.torrentFiles) {
          this.torrentFiles[index].selected = false
        }
      }
    }
  }
</script>

<style lang="scss">
  .upload-torrent {
    width: 100%;
    .el-upload, .el-upload-dragger {
      width: 100%;
    }
    .el-upload-dragger {
      border-radius: 4px;
      padding: 24px;
      height: auto;
    }
    .upload-inbox-icon {
      display: inline-block;
      margin-bottom: 12px;
    }
    .torrent-name {
      margin-top: 4px;
      font-size: $--font-size-small;
      color: $--color-text-secondary;
      line-height: 16px;
    }
  }
  .torrent-files{
    margin-top: 10px;
    border: 1px dashed #d9d9d9;
    border-radius: 4px;
    padding: 24px;
    position: relative;
    .torrent-files-top{
      display: flex;
      position: absolute;
      top: 0;
      left: 5px;
      font-size: 12px;
      span{
        color: #5b5bea;
        cursor: pointer;
      }
    }
    ul{
      height: 80px;
      overflow-y: auto;
      padding: 0;
      margin: 0;
    }
    li{
      text-align: left;
      list-style: none;
      padding: 2px 0;
      label{
        display: flex;
        align-items: center;
      }
      .el-checkbox__label{
        display: flex;
        justify-content: space-between;
        :first-child{
          width: 400px !important;
          overflow: hidden;
          text-overflow:ellipsis;
          white-space: nowrap;
        }
      }
      .chk{
        color: #BDBDBD;
      }
      .is-checked{

      }
    }
  }
</style>
