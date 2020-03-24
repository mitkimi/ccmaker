<template>
  <div class="page">
    <div class="menu">
      <el-menu
      default-active="about">
      <el-menu-item v-for="item,index in data" :key="index" :index="item.index" @click="handleSelectMenu(index)">
        <i :class="item.icon"></i>
        <span slot="title">{{item.name}}</span>
      </el-menu-item>
      <el-menu-item @click="handleBack">
        <i class="el-icon-arrow-left"></i>
        <span slot="title">返回</span>
      </el-menu-item>
    </el-menu>
    </div>
    <div class="main">
      <div class="content">
        <div class="about" v-if="current === 0">
          <h1>关于</h1>
          <div><img src="../../main/assets/logo.png" style="width: 120px;" /></div>
          <p>开源软件，免费使用，<span style="color: #ff0000">禁止售卖</span>。</p>
          <h1>版本</h1>
          <p>当前版本：<span style="color: #ff9900">0.0.1</span></p>
          <p>初代版本，可能有很多使用上面并不方便。欢迎多提宝贵意见</p>
          <p>请发送邮件到：i@mitkimi.com</p>
          <p>或访问：www.mitkimi.com</p>
          <h1>研发</h1>
          <p>github地址：https://github.com/mitkimi/ccmaker</p>
          <p>欢迎pull request</p>
        </div>
        <div class="help" v-if="current === 1">
          <h1>帮助</h1>
          <p>首次使用，请关注此页面，使用方法如下。</p>
          <h2>观看帮助视频</h2>
          <p>bilibili https://www.bilibili.com/video/BV157411U7Wd/</p>
          <p>youtube https://youtu.be/Cts8tiAknhs</p>
          <p>百度云 链接: https://pan.baidu.com/s/1fagrdB0LK-88PJyrRXi_Nw  密码: erl0</p>
        </div>
        <div class="donate" v-if="current === 2">
          <h1>赞助</h1>
          <p>如果我开发的软件对你有帮助，能否请我喝一杯咖啡？</p>
          <h2>微信支付</h2>
          <div>
            <img src="../assets/2851086542.jpg" style="width: 240px;" />
          </div>
          <h2>支付宝付款</h2>
          <div>
            <img src="../assets/3913392008.jpg" style="width: 240px;" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      current: 0,
      data: [
        {
          name: '关于',
          index: 'about',
          icon: 'el-icon-warning-outline'
        },
        {
          name: '帮助',
          index: 'help',
          icon: 'el-icon-paperclip'
        },
        {
          name: '赞助',
          index: 'donate',
          icon: 'el-icon-shopping-bag-1'
        }
      ]
    }
  },
  methods: {
    handleSelectMenu (index) {
      this.current = index
    },
    handleBack () {
      this.$router.push({
        path: '/'
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.page {
  height: 100vh;
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  .menu {
    width: 200px;
    background: #efeff3;
  }
  .main {
    width: calc(100vw - 200px);
    overflow-y: scroll;
    .content {
      margin: 10px 20px;
      h1 {
        font-size: 24px;
        height: 40px;
        line-height: 40px;
        border-bottom: 1px solid #efeff3;
        margin-top: 10px;
        margin-bottom: 10px;
      }
      h2 {
        font-size: 18px;
        height: 40px;
        line-height: 40px;
        border-bottom: 1px solid #efeff3;
        margin-top: 10px;
        margin-bottom: 10px;
      }
      p {
        color: #666666;
        line-height: 30px;
      }
    }
  }
}
</style>
