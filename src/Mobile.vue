<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 40,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `/*
* 大家好，我是帅气迷人的CTF超级老赛棍aa师傅
* 快开学了，也要准备社团招新啦
* 你有没有接触计算机安全的想法呀
* 如果有的话，不妨继续看下去
*/

/* 首先给所有元素加上过渡效果 */
* {
  transition: all .3s;
}
/* 白色背景太单调了，我们来点背景 */
html {
  color: rgb(222,222,222);
  background: #808e9b;
}
/* 文字离边框太近了 */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  overflow: auto;
  width: 90vw;
  margin: 2.5vh 5vw;
  height: 90vh;
}
/* 太高了 */
.styleEditor {
  height: 45vh;
}
/* 代码高亮 */
.token.selector{
  color: #05c46b;
}
.token.property{
  color: #575fcf;
}
.token.punctuation{
  color: yellow;
}
.token.function{
  color: #ef5777;
}

/* 加点 3D 效果呗 */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 0;
  transform: rotateX(-10deg) translateZ(-50px) ;
}

/* 接下来我给自己准备一个编辑器 */
.resumeEditor{
  position: fixed;
  top: 50%; left: 0;
  padding: .5em;  margin: 2.5vh;
  width: 95vw; height: 45vh;
  border: 1px solid;
  font-family: 华文细黑;
  background: #1e272e;
  color: #ff3f34;
  overflow: auto;
}
/* 好了，我开始介绍了 */

`,

          `
/* 好像差点什么
 * 对了，这是 Markdown 格式的
 * 简单，用开源工具翻译成 HTML 就行了
 */
`
          ,
          `
/* 再对 HTML 加点样式 */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `
# SUS2019招新
![](http://chuantu.xyz/t6/702/1564538209x1031866013.jpg)
> 秉承古典黑客精神，引领一流网络安全体验

---
### 基本情况
东南大学网络安全联盟成立于2005年，成员囊括了各级研究生与本科生
所拥有的实验室位于九龙湖校区电子电工楼。同时社团作为学校CTF校队与新成立的网络安全学院联系密切
举办了两届江苏省网络安全精英邀请赛。

---
### 社团活动
目前社团的主要活动为参与国内外的CTF竞赛
最近获得了2019年全国大学生信息安全竞赛实践赛全国一等奖

CTF也在最近火热的网剧中作为背景
还有一大波可爱的妹子在网络平台上询问“如何找一个打CTF的男朋友”
CTF可以简单的理解为现实中网络攻防的简化版
其使枯燥而且存在运气成分的安全攻防变得充满游戏性。

那么，如果你对CTF充满兴趣或者以前便接触过CTF，SUS诚邀您的加入。

---
### 招新方法
开学后为新人开设由社团成员主讲的CTF各个方向系列课程
之后举办题目相对基础的招新赛，邀请成绩优异的新人加入。

具体的内容请您持续后续情况

---
社团博客: [blog](https://susers.github.io)


社团招新qq群（务必加入）：
![](http://chuantu.xyz/t6/702/1564543612x1031866013.jpg)

---
那么
**欢迎来到hack的世界**

`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          this.$nextTick(() => {
            this.$refs.resumeEditor.goTop()
          })
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    min-height: 100vh; position: relative;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }

</style>
