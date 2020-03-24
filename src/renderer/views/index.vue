<template>
  <div class="page">
    <div class="header">
      <div class="logo"><img src="../assets/cc.png" /> 字幕编辑器</div>
      <div class="operation">
        <el-button @click="handleShowAbout">关于</el-button>
      </div>
    </div>
    <div class="body">
      <div class="has-video" v-show="video">
        <div class="video-container section">
          <div class="title">
            <span>视频预览</span>
            <div class="extra">
              字幕预览：
              <el-switch
                v-model="isViewingSubtitle"
                active-color="#13ce66">
              </el-switch>
            </div>
          </div>
          <div class="video">
            <video :src="video"
              ref="video"
              controls
              @timeupdate="canViewSubtitle"
              style="width: calc(100vw - 600px - 40px); height: calc((100vw - 600px - 40px) * (9/16)); position: absolute; top: 0; left: 0; z-index: 1;"
              ></video>
            <div v-for="(item,index) in cc.body" :key="index" v-if="isViewingSubtitle && index === viewingIndex" class="sub-title-viewer" ref="subtitleViewer"
              :style="'background: rgba(0,0,0,.8);'">
              <!-- :style="'background: ' + cc.background_color + '; opacity: ' + cc.background_alpha + ';'"> -->
              {{item.content}}
            </div>
          </div>
          <!-- <div class="control-bar">
            <div class="process-bar">
              <div class="loaded" style="width: 80%"></div>
              <div class="played" :style="'width:'+calcPlayedProgress()"></div>
            </div>
            <div class="controls">
              <div class="button-container">
                <span class="button" @click="handelPlayingToggle">
                  <i class="el-icon-video-play" v-if="!player.playing"></i>
                  <i class="el-icon-video-pause" v-else></i>
                </span>
              </div>
              <div class="button-container"></div>
            </div>
          </div> -->
          <!-- <div class="title">字幕预览</div>
          <div class="sub-title-view">
            <div class="sub-title">正在显示的字幕</div>
          </div> -->
        </div>
        <div class="video-subtitle-editor section">
          <div class="title">
            <span>字幕</span>
            <div class="extra">
              <el-button v-if="subtitle.mode !== 'text'" size="mini" style="margin-right: 10px;" @click="handleSwitchMode('text')">多行编辑模式</el-button>
              <el-button v-if="subtitle.mode === 'text'" size="mini" style="margin-right: 10px;" @click="handleReadTxtFile">载入txt多行文件</el-button>
            </div>
          </div>
          <div class="textarea-container" v-if="subtitle.mode === 'text'">
            <div class="tips">将每条字幕占一行键入下面的文本框里，将进行自动的拆分</div>
            <el-input type="textarea" style="height: calc(100vh - 240px);" v-model="subtitle.text"></el-input>
            <div style="margin-top: 10px;">
              <el-button type="primary" @click="handleCreateSubtitle">生成带时间的字幕</el-button>
            </div>
          </div>
          <div class="sub-title-container" id="sub-title-container" v-if="subtitle.mode === 'view'">
            <div class="sub-title-item" v-for="(item,index) in cc.body" :key="index" :id="'subtitleIndex_'+index" :ref="'subtitleIndex_'+index">
              <div class="sub-title">
                <div class="time">
                  <div class="from" @click="handleQueryCurrentTime(index, 'from')">{{item.from}}</div>
                  <div class="to" @click="handleQueryCurrentTime(index, 'to')">{{item.to}}</div>
                </div>
                <div class="content">{{item.content}}</div>
              </div>
              <transition name="el-fade-in">
                <div class="current" v-if="index === viewingIndex"></div>
                <div class="next" v-if="index === current + 1"></div>
              </transition>
            </div>
          </div>
          <div class="title" style="margin-top: 10px;" v-if="subtitle.mode === 'view'">生成字幕</div>
          <div class="options" style="padding: 0 10px;" v-if="subtitle.mode === 'view'">
            <el-button @click="handleCreateBilibiliCCFile">生成b站 cc字幕</el-button>
            <el-button @click="handleCreateYouTubeCCFile">生成youTube cc字幕</el-button>
          </div>
        </div>
      </div>
      <div class="none-video" v-show="!video">
        <svg t="1584807939303" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="2763" width="200" height="200"><path d="M960 192l-28.384 0c-16.8 0-32.928 6.624-44.928 18.432l-86.688 85.504 0-39.936c0-53.024-43.008-96-96-96l-608 0c-52.928 0-96 43.04-96 96l0 512c0 52.992 42.976 96 96 96l608 0c52.992 0 96-43.008 96-96l0-39.072 86.688 85.504c12 11.808 28.128 18.432 44.928 18.432l28.384 0c35.328 0 64-28.64 64-64l0-512.864c0-35.36-28.672-64-64-64zM96 800c-17.664 0-32-14.368-32-32l0-512c0-17.696 14.304-32 32-32l608 0c17.632 0 32 14.336 32 32l0 512c0 17.632-14.368 32-32 32l-608 0zM960 768.864l-32 0-128-128 0-0.864-32-32 0-192 160-160 32 0 0 512.864z" p-id="2764" fill="#1296db"></path></svg>
        <el-button type="primary" @click="$refs.file.click()">加载视频</el-button>
        <div class="text">请先加载视频，仅支持mp4、m4v、mov、ogg格式，本视频不上传。</div>
        <input type="file" id="file" ref="file" style="display: none;" @change="loadFile" />
      </div>
    </div>
  </div>
</template>

<script>
// const { shell } = require('electron')
import fs from 'fs'
// import moment from 'moment'
const DemoBcc = require('./subtitle_demo.bcc.json')
export default {
  data () {
    return {
      player: {
        playing: false
      },
      subtitle: {
        mode: 'text', // text: 默认，多行编辑模式， view: 预演模式
        text: ``
      },
      isViewingSubtitle: true,
      video: '',
      cc: DemoBcc,
      current: -1,
      viewingIndex: -1,
      timer: null
    }
  },
  mounted () {
    clearInterval(this.timer)
    this.timer = setInterval(() => {
      // this.canViewSubtitle(this.current, this.viewingIndex)
    }, 100)
  },
  methods: {
    handleReadTxtFile () {
      // 载入多行字幕文件
      const inputObj = document.createElement('input')
      inputObj.setAttribute('id', 'txtFile')
      // inputObj.setAttribute('ref', 'saver')
      inputObj.setAttribute('type', 'file')
      inputObj.setAttribute('style', 'visibility: hidden')
      document.body.appendChild(inputObj)
      inputObj.addEventListener('change', this.readTxtFile)
      inputObj.click()
    },
    readTxtFile () {
      const ef = document.getElementById('txtFile')
      var files = ef.files
      const path = files[0].path
      console.log('path', path, files[0])
      fs.readFile(path, (err, data) => {
        console.log('callback', err, data)
        if (!err) {
          this.subtitle.text = data.toString()
        }
      })
      // this.subtitle.text = txt
    },
    handleCreateYouTubeCCFile () {
      const inputObj = document.createElement('input')
      inputObj.setAttribute('id', 'y_ef')
      // inputObj.setAttribute('ref', 'saver')
      inputObj.setAttribute('type', 'file')
      inputObj.setAttribute('style', 'visibility: hidden')
      inputObj.setAttribute('webkitdirectory', '')
      inputObj.setAttribute('directory', '')
      document.body.appendChild(inputObj)
      inputObj.addEventListener('change', this.saveYoutubeToDirectory)
      inputObj.click()
    },
    handleCreateBilibiliCCFile () {
      const inputObj = document.createElement('input')
      inputObj.setAttribute('id', 'b_ef')
      // inputObj.setAttribute('ref', 'saver')
      inputObj.setAttribute('type', 'file')
      inputObj.setAttribute('style', 'visibility: hidden')
      inputObj.setAttribute('webkitdirectory', '')
      inputObj.setAttribute('directory', '')
      document.body.appendChild(inputObj)
      inputObj.addEventListener('change', this.saveBilibiliToDirectory)
      inputObj.click()
    },
    saveBilibiliToDirectory () {
      // const ef = this.$refs.saver
      const ef = document.getElementById('b_ef')
      var files = ef.files
      const path = files[0].path
      const filename = new Date() * 1
      let jsonObj = {}
      const filetype = 'bcc'
      console.log(files)
      // 写入文件
      jsonObj = JSON.stringify(this.cc)
      // 生成文件
      fs.writeFile(`${path}/${filename}.${filetype}`, jsonObj, (err) => {
        if (err) {
          this.$message.error(err)
        } else {
          this.$message.success('文件保存成功')
        }
      })
    },
    saveYoutubeToDirectory () {
      // const ef = this.$refs.saver
      const ef = document.getElementById('y_ef')
      var files = ef.files
      const path = files[0].path
      const filename = new Date() * 1
      let jsonObj = {}
      const filetype = 'srt'
      let arr = []
      this.cc.body.map(subtitle => {
        console.log(subtitle)
        const from = this.secondToStandardTime(subtitle.from)
        const to = this.secondToStandardTime(subtitle.to)
        const content = subtitle.content
        const cct = [`${from},${to}`, content, '']
        arr.push(cct)
      })
      let str = ''
      arr.map(cc => {
        cc.map(line => {
          str += `${line}\n`
        })
      })
      jsonObj = JSON.stringify(str)
      console.log(jsonObj)
      // 生成文件
      fs.writeFile(`${path}/${filename}.${filetype}`, str, (err) => {
        if (err) {
          this.$message.error(err)
        } else {
          this.$message.success('文件保存成功')
        }
      })
    },
    secondToStandardTime (value) {
      let second = 0
      let minute = 0
      let hour = 0
      let last = value // 秒
      if (last >= 0 && last < 60) {
        second = last
        last = 0
      } else {
        second = last % 60
        last = parseInt(last / 60) // 分
      }
      // console.log('min', last)
      if (last >= 0 && last < 60) {
        minute = last
        last = 0
      } else {
        minute = last % 60
        last = parseInt(last / 60) // 时
      }
      // console.log('hour', last)
      // if (last >= 0 && last < 24) {
      //   hour = last
      //   last = 0
      // } else {
      //   hour = last % 24
      //   last = parseInt(last / 24) // 天
      // }
      hour = last

      // console.log(value, day, hour, minute, second)
      if (value >= 0) {
        return `${hour}:${minute}:${second}`
      } else {
        return '0:00:00.000'
      }
    },
    handelPlayingToggle () {
      if (this.player.playing) {
        this.$refs.video.pause()
        this.player.playing = false
      } else {
        this.$refs.video.play()
        this.player.playing = true
      }
    },
    handleSwitchMode (mode) {
      this.subtitle.mode = mode
    },
    loadFile () {
      // console.log(arguments)
      const selectedFile = this.$refs.file.files[0]
      // const name = selectedFile.name // 选中文件的文件名
      // const size = selectedFile.size // 选中文件的大小
      // const file = selectedFile.path
      this.video = `${selectedFile.path}`
      console.log('video', this.$refs.video)
      // console.log('file', selectedFile)
      // fs.ReadStream(selectedFile.path, () => {
      //   console.log('fsreadstreamreturn', arguments)
      // })
      // this.$refs.video.src = selectedFile.path
    },
    canViewSubtitle () {
      // const current = this.current
      // const viewingIndex = this.viewingIndex
      // 检测正在预览哪一条
      const videoNowAt = this.$refs.video.currentTime
      const subtitle = this.cc.body
      for (let i = 0; i < subtitle.length; i++) {
        if (i === -1) {
          this.current = -1
          this.viewingIndex = -1
        } else if (videoNowAt >= subtitle[i].from && videoNowAt < subtitle[i].to) {
          this.current = i
          this.viewingIndex = i
          this.scrollTo(i)
          break
        } else {
          this.viewingIndex = null
        }
      }

      this.calcPlayedProgress()
    },
    scrollTo (index) {
      const top = document.getElementById([`subtitleIndex_${index}`]).offsetTop - 200
      // console.log('字幕条滚动到top', top)
      document.getElementById('sub-title-container').scrollTo({
        top,
        behavior: 'smooth'
      })
    },
    handleCreateSubtitle () {
      this.$confirm('在此处生成字幕时，将清除已设置好的时间节点，是否继续？', '提示', {
        confirmButtonText: '好',
        cancelButtonText: '再考虑一下',
        type: 'warning'
      }).then(() => {
        // console.log(this.subtitle.text)
        const arr = this.subtitle.text.split(/[\n]/) // 根据换行和空格把文字进行拆分
        // console.log(arr)
        const body = []
        arr.map(subtitle => {
          const obj = {
            from: null,
            to: null,
            content: subtitle,
            location: 2
          }
          body.push(obj)
        })
        this.cc.body = body
        this.current = -1
        this.viewingIndex = -1
        this.subtitle.mode = 'view'
      })
    },
    handleQueryCurrentTime (index, type) {
      const currentTime = this.$refs.video.currentTime
      console.log(this.current, currentTime)
      if (index === 0) {
        // 第一个
        if (type === 'from') {
          // 点了from
          this.cc.body[index].from = currentTime
        }
        if (type === 'to') {
          // 点了to
          this.cc.body[index].from = 0
          this.cc.body[index].to = currentTime
        }
      } else {
        // 其余的
        if (type === 'from') {
          // 点了from
          this.cc.body[index].from = currentTime
          if (!this.cc.body[index - 1].to || this.cc.body[index - 1].to > currentTime) {
            this.cc.body[index - 1].to = currentTime
          }
        }
        if (type === 'to') {
          // 点了to
          this.cc.body[index].to = currentTime
        }
        this.current = index
        this.viewingIndex = index
        this.scrollTo(index)
        if (this.current === this.cc.body.length - 1 && type === 'to') {
          this.$confirm('字幕时间添加完毕，是否回到顶部？', '提示', {
            confirmButtonText: '好',
            cancelButtonText: '暂时不回',
            type: 'success'
          }).then(() => {
            this.$refs.video.pause()
            this.$refs.video.currentTime = 0
            this.current = -1
            this.viewingIndex = -1
            document.getElementById('sub-title-container').scrollTo({
              top: 0,
              behavior: 'smooth'
            })
          })
        }
      }
    },
    calcPlayedProgress () {
      setTimeout(() => {
        const video = this.$refs.video
        const duration = video.duration
        const nowAt = video.currentTime
        console.log(nowAt / duration)
        return nowAt / duration
      }, 0)
    },
    handleShowAbout () {
      this.$router.push({
        path: 'about'
      })
    }
  }
}
</script>

<style lang="scss" src="./index.scss" scoped></style>
