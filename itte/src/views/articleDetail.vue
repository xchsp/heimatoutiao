<template>
  <div class="articaldetail">
    <div class="header">
      <div class="left">
        <van-icon name="arrow-left back" @click="$router.back()" />
        <span class="iconfont iconnew new"></span>
      </div>
      <span
        @click="followAuthor"
        :class="{active:article.has_follow}"
      >{{article.has_follow?"已关注":"关注"}}</span>
    </div>
    <div class="detail">
      <div class="title">{{article.title}}</div>
      <div class="desc">
        <span>{{article.user&&article.user.nickname}}</span> &nbsp;&nbsp;
        <span>2019-9-9</span>
      </div>
      <div class="content" v-html="article.content" v-if="article.type===1"></div>
      <video
        v-if="article.type===2"
        :src="article.content"
        controls
        autoplay
        poster="http://img4.imgtn.bdimg.com/it/u=1231851507,1502682343&fm=26&gp=0.jpg"
      ></video>
      <div class="opt">
        <span class="like" @click="like" :class="{active:article.has_like}">
          <van-icon name="good-job-o" />
          {{article.like_length}}
        </span>
        <span class="chat">
          <van-icon name="chat" class="w" />微信
        </span>
      </div>
    </div>
    <!-- 精彩跟帖 -->
    <div class="keeps">
      <h2>精彩跟帖</h2>
      <div class="item" v-for="item in commentData" :key="item.id">
        <div class="head">
          <img :src="item.user.head_img" alt />
          <div>
            <p>{{item.user.nickname}}</p>
            <span>2小时前</span>
          </div>
          <span>回复</span>
        </div>
        <div class="text">{{item.content}}</div>
      </div>
      <div class="more">更多跟帖</div>
    </div>
    <componentFooter :ishow="true" :article="article"></componentFooter>
  </div>
</template>

<script>
import { getArticleDetail, getCommentData } from '@/api/article.js'
import { followUser, unFollowUser, likeArticle } from '@/api/user.js'
import componentFooter from '@/components/component_footer.vue'

export default {
  data () {
    return {
      article: {},
      commentData: []
    }
  },
  components: {
    componentFooter
  },
  async mounted () {
    // 根据id获取文章的详情，实现文章详情的动态渲染
    let res = await getArticleDetail(this.$route.params.id)
    if (res.status === 200) {
      this.article = res.data.data
      // 获取文章评论数据
      let res1 = await getCommentData(this.article.id)
      console.log(res1)
      if (res1.status === 200) {
        this.commentData = res1.data.data.map(value => {
          // 给图片添加基地址
          value.user.head_img = localStorage.getItem('mybaseURL') + value.user.head_img
          return value
        })
      }
    }
  },
  methods: {
    async followAuthor () {
      let res
      // 判断当前状态是关注还是未关注
      if (this.article.has_follow) {
        // 已关注
        res = await unFollowUser(this.article.user.id)
      } else {
        res = await followUser(this.article.user.id)
      }
      this.$toast.success(res.data.message)
      //    this.article.has_follow取反，改变按钮状态
      this.article.has_follow = !this.article.has_follow
    },
    // 点赞与取消点赞
    async like () {
      let res = await likeArticle(this.article.id)
      // 通过has_like的取值，来决定点击之后是否添加active样式
      this.article.has_like = !this.article.has_like
      if (res.data.message === '点赞成功') {
        this.article.like_length++
      } else if (res.data.message === '取消成功') {
        this.article.like_length--
      }
      this.$toast.success(res.data.message)
    }
  }
}
</script>

<style lang='less' scoped>
.header {
  padding: 0px 10px;
  height: 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #ccc;
  > .left {
    height: 100%;
    vertical-align: middle;
    position: relative;
    .van-icon {
      position: absolute;
      top: 15px;
      left: 0;
    }
    > span {
      margin-left: 22px;
      font-size: 50px;
    }
  }
  > span {
    padding: 5px 15px;
    border: 1px solid #ccc;
    color: #333;
    text-align: center;
    border-radius: 15px;
    font-size: 13px;
  }
}
.active {
  color: #fff !important;
  background-color: #f00;
}
.detail {
  padding: 15px;
  .title {
    font-size: 18px;
    font-weight: bold;
    padding: 10px 0;
  }
  .desc {
    line-height: 30px;
    color: #999;
    font-size: 13px;
  }
  .content {
    text-indent: 2em;
    line-height: 24px;
    font-size: 15px;
    padding-bottom: 30px;
    width: 100%;
  }
  video {
    width: 100%;
    margin-bottom: 10px;
  }
}
.opt {
  display: flex;
  justify-content: space-around;
  .like,
  .chat {
    height: 25px;
    padding: 0 15px;
    font-size: 14px;
    line-height: 25px;
    text-align: center;
    border: 1px solid #ccc;
    border-radius: 15px;
  }
  .w {
    color: rgb(84, 163, 5);
  }
}
.keeps {
  border-top: 5px solid #ddd;
  padding: 0 15px 50px 15px;
  > h2 {
    line-height: 50px;
    text-align: center;
  }
  .item {
    padding: 10px 0;
    border-bottom: 1px solid #ccc;
    .head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      > img {
        width: 50/360 * 100vw;
        height: 50/360 * 100vw;
        display: block;
        border-radius: 50%;
      }
      > div {
        flex: 1;
        display: flex;
        flex-direction: column;
        margin-left: 10px;
        > span {
          font-size: 12px;
          color: #999;
          line-height: 25px;
        }
      }
      > span {
        color: #999;
        font-size: 13px;
      }
    }
    .text {
      font-size: 14px;
      color: #333;
      padding: 20px 0 10px 0;
    }
  }
  .more {
    width: 100px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    border: 1px solid #ccc;
    border-radius: 15px;
    margin: 20px auto;
    font-size: 13px;
  }
}
/deep/.photo {
  img {
    width: 100% !important;
    display: block;
  }
}
</style>
