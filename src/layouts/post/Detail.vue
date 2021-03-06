<template>
  <article v-loading="loading">
    <div class="container">
      <main>
        <div class="content">
          <div class="title">{{ payload.title }}</div>
          <div class="stuff">
            <span>{{ formatTime(payload.time) }}</span>
            <span>字数{{ count }}</span>
            <span>作者：{{ payload.author }}</span>
          </div>
          <hr class="line">
          <markdown-editor ref="content" :content="payload.content" :highlight="highlight"></markdown-editor>
        </div>
        <div class="comment">
          <div class="point">
            <i class="iconfont icon-pinglun1"></i> 写评论
          </div>
          <comment-input @onClick="submitComment"></comment-input>
          <div class="point">
            <i class="iconfont icon-pinglun"></i> 评论区
          </div>
          <comment-box v-if="commentList.length > 0" v-for="(item, index) in commentList" :key="index">
            <p slot="header" class="title">
              <span class="name">{{ item.name }}</span>
              <span class="time">{{ item.time.substring(0, 10) }}</span>
            </p>
            <div slot="content">{{ item.content }}</div>
          </comment-box>
          <p v-if="commentList.length === 0">暂无评论信息</p>
        </div>
      </main>
    </div>
  </article>
</template>

<script>
import MarkdownEditor from '@/components/MarkdownEditor'
import CommentBox from '@/components/blog/CommentBox'
import CommentInput from '@/components/blog/CommentInput'
import Prism from 'prismjs'
export default {
  components: {
    MarkdownEditor,
    CommentBox,
    CommentInput
  },
  data () {
    return {
      loading: false,
      payload: {
        title: '',
        content: '',
        time: ''
      },
      commentList: [],
      count: 0,
      month: ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月']
    }
  },
  mounted () {
    this.getPostInfo()
    this.getCommentInfo()
  },
  computed: {
  },
  methods: {
    getPostInfo () {
      this.loading = true
      this.$request.post({
        url: api => api.post.read,
        params: {
          id: this.$route.params.id
        }
      }).then(response => {
        let body = response.body
        if (body.code === 1) {
          this.payload = { ...body.payload }
          this.$nextTick(() => {
            this.count = this.$refs.content.$el.innerText.length
          })
        }
        this.loading = false
      })
    },
    getCommentInfo () {
      this.loading = true
      this.$request.get({
        url: api => api.comment.read,
        params: {
          arcticle_id: this.$route.params.id
        }
      }).then(reslut => {
        this.commentList = reslut.body.payload
        this.loading = false
      })
    },
    submitComment (options) {
      this.loading = true
      let articleId = this.$route.params.id
      this.$request.post({
        url: api => api.comment.create,
        params: {
          arcticle_id: articleId,
          ...options
        }
      }).then(response => {
        this.loading = false
        if (response.body.code === 1) {
          // 重新加载评论页面
          this.getCommentInfo()
          // window.toast('创建成功')
        } else {
          window.alert('创建失败')
        }
      })
    },
    formatTime (time) {
      let date = time.substring(8, 10)
      let month = this.month[parseInt(time.substring(5, 7)) - 1]
      let year = time.substring(0, 4)
      return `${month} ${date}，${year}`
    },
    highlight (data) {
      let code = Prism.highlight(data, Prism.languages.javascript, 'javascript')
      return code
    }
  }
}
</script>

<style lang="less" scoped>
img {
  width: 100% !important;
}
.container {
  height: 100%;
  overflow-y: scroll;
}
main {
  box-sizing: border-box;
  width: 950px;
  margin: 30px auto;
  .content {
    background-color: rgba(255, 255, 255, 0.95);
    border-radius: 3px;
    padding: 30px 50px 50px;
    color: #666;
    line-height: 2;
    font-size: 1em;
    .title {
      font-size: 30px;
      color: #333;
      line-height: 1.3;
      position: relative;
      font-weight: bold;
      font-family: BankGothicLight;
    }
    .stuff {
      margin-top: 20px;
      color: #666;
      font-size: 13px;
      span {
        margin-right: 20px;
        display: inline-block;
      }
    }
    .line {
      width: 200px;
      margin: 25px 0 35px;
      line-height: 1;
      color: #eee;
    }
  }
  .comment {
    background-color: rgba(255, 255, 255, 0.95);
    margin: 20px 0 80px;
    padding: 20px 40px;
    .point {
      font-size: 16px;
      line-height: 2;
      margin-bottom: 10px;
      color: #555;
      i {
        color: #555;
      }
    }
    .title {
      position: relative;
      .time {
        position: absolute;
        top: 5px;
        right: 0;
        font-size: 13px;
        color: #888;
      }
    }
  }
}
</style>

<style scoped lang="css">
article {
  position: relative;
  width: 100%;
  height: calc(100% - 50px);
  overflow: hidden;
  box-sizing: border-box;
  background: url('https://images-1254073471.cos.ap-shanghai.myqcloud.com/sky.jpg') 100% no-repeat;
}
</style>
