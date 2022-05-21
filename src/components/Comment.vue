<template>
  <div id="app-container">

    <div :class="navBarFixed == true ? 'navBarWrap' :''">
      <sidebar class="sidebar-container"/>

      <div>
        <el-form :inline="true" :model="formData" class="demo-form-inline">
          <el-form-item label="文章ID">
            <el-input v-model="formData.cid" placeholder="100056701"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="success" @click="getArticles">查询</el-button>
          </el-form-item>
          <el-button @click="drawer = true" type="success" id="btn_article">章节</el-button>

          <el-button @click="nextComments" type="success" style="margin-left: 3%">NEXT</el-button>
          <el-switch style="display: inline; margin-left: 40%"
                     v-model="showAll"
                     active-color="#13ce66"
                     inactive-color="#ff4949"
                     inactive-text="显示所有"
                     active-text="只看作者">
          </el-switch>
        </el-form>
      </div>
    </div>


    <el-drawer :visible.sync="drawer" :with-header="false">
      <el-table class="pointer" :data="articleList" height="90%" border style="width: 100%">
        <el-table-column>
          <template slot-scope="scope">
            <a class="article" v-html="scope.row.article_title" @click="showComments(scope.row.id)"></a>
          </template>
        </el-table-column>
      </el-table>
    </el-drawer>

    <hr/>

    <!--    <template>-->
    <!--      <el-button @click="resetDateFilter">清除日期过滤器</el-button>-->
    <!--      <el-button @click="clearFilter">清除所有过滤器</el-button>-->
    <!--      <el-table-->
    <!--          ref="filterTable"-->
    <!--          :data="commentList"-->
    <!--          style="width: 100%">-->
    <!--        <el-table-column-->
    <!--            prop="comment_content"-->
    <!--            label="内容"-->
    <!--            sortable-->
    <!--            width="auto"-->
    <!--            column-key="date"-->
    <!--            :filters="[{text: '2016-05-01', value: '2016-05-01'}, {text: '2016-05-02', value: '2016-05-02'}, {text: '2016-05-03', value: '2016-05-03'}, {text: '2016-05-04', value: '2016-05-04'}]"-->
    <!--            :filter-method="filterHandler"-->
    <!--        >-->
    <!--        </el-table-column>-->
    <!--      </el-table>-->
    <!--    </template>-->

    <template>
      <div class="ul_css" style="overflow:auto">

        <ul>
          <li v-for="item in commentList" :key="item.id">
            <div>
              <span class="li_css" v-html="item.comment_content"></span>
              <br>
              <br>
              <div class="reply_css" v-for="reply in item.replies" :key="reply.id">
                    <span v-html="reply.content">
                    </span>
              </div>

              <br>
              <br>

              <hr>
              <br>

            </div>
          </li>
        </ul>
      </div>
    </template>


  </div>
</template>

<script>
import axios from "axios";
//import request from "@/utils/http/request.js";

export default {
  name: "Comment",
  mounted() {
    window.addEventListener('scroll', this.watchScroll);
  },

  data() {
    return {
      navBarFixed: false,
      showAll: false,
      count: 10,
      // column info
      formData: {
        cid: "100056701",
        size: 500,
        prev: 0,
        order: "earliest",
        sample: false,
      },
      // article data of column
      articleList: [
        {
          id: 0,
          article_title: "",
        },
      ],
      currentArticleId: 0,
      previousCommentId: 0,
      // article from to send
      articleFrom: {
        aid: 0,
        prev: 0
      },
      authorizedReply: '作者回复',
      commentList: [
        {
          id: 0,
          score: "",
          comment_content: "",
          discussion_count: 0,
          replies: [
            {
              id: 0,
              content: "",
              user_name: ""
            }
          ]
        }
      ],
      drawer: false,
      // disabled infinite-scroll
      disabledScroll: true,
    };
  },
  created() {
    this.commentList = [];
  },
  methods: {
    watchScroll() {
      var scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
      //  当滚动超过 50px 时，实现吸顶效果(滚动条高度当前设置为：50px)
      if (scrollTop > 49) {
        this.navBarFixed = true;
      } else {
        this.navBarFixed = false;
      }
    },
    getArticles() {
      axios.post("/api/serv/v1/column/articles", this.formData).then((resp) => {
        this.articleList = resp.data.data.list;
      });
      this.drawer = true
    },
    load() {
      this.$message({
        message: '加载中..'
      })
      this.showComments(this.currentArticleId)
    },
    nextComments() {
      this.commentList = [];
      this.showComments(this.currentArticleId)
    },
    async showComments(currentArticleId) {
      await this.doShowComments(currentArticleId);
    },
    async doShowComments(currentArticleId) {

      let newArticle = currentArticleId !== this.currentArticleId;
      this.articleFrom.aid = this.currentArticleId = currentArticleId
      if (newArticle) {
        this.articleFrom.prev = 0
      }
      {
        this.articleFrom.prev = this.previousCommentId
      }
      await axios.post("/api/serv/v1/comments", this.articleFrom).then((resp) => {


        this.commentList = resp.data.data.list
        this.previousCommentId = this.commentList[this.commentList.length - 1].score

        if (!this.showAll) {
          this.commentList = this.commentList.filter((comment) => {
            return comment.discussion_count !== 0
          });
        }
        if (this.commentList === undefined || this.commentList === null || this.commentList === []) {
          this.$message({
            message: '当前页作者未回复'
          })
        }

        if (resp.data.data.page.more === false) {
          this.$message({
            message: '已经到底了...',
            type: 'warning'
          })
        }

      })


    }
  },
};
</script>

<style scoped>
.navBarWrap {
  position: fixed;
  top: 0;
  padding-top: 3%;
  background: #fff;
  /*z-index: 9;*/
  width: 100%;
}

#app-container {
  margin: 2% 3%;
}

#btn_article {
  margin-left: 3%;
}

.article {
  color: cadetblue;
  line-height: 10px;
  margin: 3%;
  font-size: 15px;
}

#drawer_box {
  margin: 3%;
}

.pointer {
  cursor: pointer;
}

a:hover {
  color: #ff4f46;
}

ui {

}

.div_css {
  margin: 8%;
  margin-bottom: 21%;
}

.btn_css {
  float: left;
  margin-top: 2%;
  margin-left: 2%;
}

.p_css {
  text-align: center;
  color: #ff4f46;
}

.ul_css {
  width: auto;
  margin-right: 2%;
  overflow: auto;
}

.li_css {
}

.reply_css {
  color: #ff4f46;
}
</style>
