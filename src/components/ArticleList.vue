<template>

  <el-scrollbar style="height: 100%; width: 250px; min-width: 250px;" ref="scrollbar" class="el-scrollbar">
    <div class="article-list" v-if="posts.length > 0">
      <div class="article-list-panel" v-for="post in posts" :key="post.id" ref="post" @click="selected(post.id)"
           :data-id="post.id" v-bind:class="{active: post.id === selectedPostId}" :id="post.id">
        <div class="article-list-item">
          <h4 class="article-title">{{ post.title }}</h4>
          <div class="article-tags">
            <span class="tag" v-for="tag in post.tags" :key="tag.name">#{{ tag.name }}</span>
          </div>
          <span class="article-date">{{ post.date }}</span>

          <a class="article-edit-btn" @click="editPost(post.id)">
            <i class="el-icon-edit-outline"></i>
          </a>
          <a class="article-delete-btn" @click="deletePost(post.id)">
            <i class="el-icon-delete"></i>
          </a>
        </div>
      </div>
    </div>
    <div class="article-none-panel" v-if="posts.length === 0">
      {{$t('noArticleMsg')}}
    </div>
  </el-scrollbar>

</template>

<script>
  import ClientAnalytics from '@/plugins/analytics'

  export default {
    data () {
      return {
        scrollWrap: null // 滚动条容器
      }
    },

    mounted () {
      this.scrollWrap = this.$refs.scrollbar.$refs.wrap // 滚动条容器，详见：el-scrollbar组件源码
      this.onscroll()
      this.setScrollLocation()
    },

    methods: {
      selected: function (selectedId) {
        this.$store.dispatch('Hexo/selectPost', selectedId)
        ClientAnalytics.event('article', 'view')
      },

      editPost: function (id) {
        this.$router.push({name: 'edit', params: {postId: id}})
      },

      async deletePost (id) {
        if (confirm(this.$t('deleteArticleConfirmMsg'))) {
          await this.$store.dispatch('Hexo/deletePost', id)
          this.$notify({title: this.$t('successTitle'), message: this.$t('deleteSuccessMsg'), type: 'success'})
          ClientAnalytics.event('article', 'delete')
        }
      },

      // 监听滚动状态，并存储滚动位置
      onscroll (e) {
        var me = this
        me.scrollWrap.addEventListener('scroll', function (e) {
          me.$store.dispatch('UiStatus/onscroll', {
            scrollTop: e.srcElement.scrollTop,
            scrollLeft: e.srcElement.scrollLeft
          })
        })
      },

      // 设置滚动位置
      setScrollLocation () {
        let scrollTop = this.$store.state.UiStatus.postListScrollTop || 0
        let scrollLeft = this.$store.state.UiStatus.postListScrollLeft || 0
        if (scrollTop > 0 || scrollLeft > 0) {
          this.scrollWrap.scrollTop = scrollTop
          this.scrollWrap.scrollLeft = scrollLeft
        }
      }
    },

    computed: {
      posts () {
        return this.$store.getters['Hexo/filteredPosts']
      },
      selectedPostId () {
        let selectedPost = this.$store.getters['Hexo/selectedPost']
        return selectedPost ? selectedPost._id : null
      }
    }
  }
</script>

<style scoped>
  .el-scrollbar {
    width: 380px;
    border-right: 1px solid #E9E9E9;
  }

  .el-scrollbar__wrap {
    /*scroll-behavior: smooth;*/
  }

  .article-list {
    background-color: #FAFAFA;
  }

  .article-none-panel {
    color: #727272;
    text-align: center;
    margin-top: 20px;
    size: 24px;
  }

  .article-list-panel.active {
    background-color: #EEEEEE;
  }

  .article-list-item {
    color: #727272;
    font-size: 12px;
    padding: 8px;
    position: relative;
    cursor: pointer;
    /*border-bottom: 1px solid rgba(235, 238, 245, 0.8);*/
  }

  .article-list-item .article-title {
    color: #323232;
    margin: 0px 0px 5px;
    font-weight: 400;
    font-size: 15px;
    width: auto;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    word-wrap: normal;
    word-wrap: break-word;
    word-break: break-all;
  }

  .article-list-item .article-tags {
    margin: 0px 0px 5px;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
  }

  .article-list-item .article-tags .tag {
  }

  .article-list-item .article-tags .tag:after {
    content: " ";
  }

  .article-list-item .article-date {
    margin: 0px 0px 5px;
  }

  .article-edit-btn {
    top: 5px;
    right: 6px;
    opacity: 0;
    position: absolute;
    padding: 5px 6px;
    border-radius: 15px;
    cursor: pointer;
    font-size: 14px;
    line-height: 1;
    transition: opacity 0.3s ease, background-color 0.1s ease;
    background-color: #d5d5d5;
  }

  .article-delete-btn {
    top: 5px;
    right: 36px;
    opacity: 0;
    position: absolute;
    padding: 5px 6px;
    border-radius: 15px;
    cursor: pointer;
    font-size: 14px;
    line-height: 1;
    transition: opacity 0.3s ease, background-color 0.1s ease;
    background-color: #d5d5d5;
  }

  .article-edit-btn:hover, .article-delete-btn:hover {
    background-color: #c0c0c0;
  }

  .article-list-panel:hover .article-edit-btn,
  .article-list-panel:hover .article-delete-btn {
    opacity: 1;
  }

</style>

