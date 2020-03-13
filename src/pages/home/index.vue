<template>
  <div class="home-container">
    <div style="padding: 20px;">
      <div>
        <el-select v-model="value" @change="checkCate" placeholder="类别推荐">
          <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </div>
      <div class="search">
        <el-input clearable v-model="search" placeholder="输入要搜索的内容"></el-input>
        <el-button @click="searchVideo" type="primary">搜索</el-button>
      </div>
      <div class="author" @click="drawer = true">
        <p>点</p>
        <p>我</p>
        <p>查</p>
        <p>看</p>
        <p>说</p>
        <p>明</p>
      </div>
    </div>
    <div class="list" v-loading="loading" element-loading-background="rgba(255, 255, 255, 0)">
      <ul v-show="!loading">
        <li
          :class="[isCheck == index ? 'check' : '']"
          @click="toPlay(item.id, index)"
          v-for="(item, index) in listData"
          :key="item.id"
        >
          <p>
            <span>{{ item.title }}</span>
          </p>
        </li>
        <li v-if="noData">没有找到你想要的影片</li>
      </ul>
    </div>

    <div class="page" v-show="listData.length">
      <el-pagination
        @current-change="changePage"
        :small="true"
        prev-text="上一页"
        next-text="下一页"
        layout="prev, pager, next"
        :page-size="Number(15)"
        :total="pageInfo.total"
        :current-page="pageInfo.page"
      ></el-pagination>
    </div>

    <el-drawer :visible.sync="drawer" :direction="direction">
      <div class="block">
        <el-carousel height="5rem">
          <el-carousel-item v-for="item in movieImg" :key="item">
            <img width="100%" height="100%" :src="item" alt />
          </el-carousel-item>
        </el-carousel>
      </div>
      <div>
        <p style="margin-top: 10px;margin-bottom: 10px;text-align: center">该站完成时间：2020年3月13日，后续不再更新</p>
        <p style="margin-bottom: 10px;text-align: center;color: red">部分视频可能无法播放，请换一个视频看</p>
        <p style="margin-bottom: 10px;text-align: center;color: red">不要相信视频中的光谷</p>
        <p style="text-align: center">番茄出品，必属精品。</p>
        <p style="margin-top: 10px;margin-bottom: 10px;text-align: center">纵有千古，横有八荒，前途似海，来日方长。</p>
      </div>
    </el-drawer>

    <el-drawer size="100%" :visible.sync="player" @close="closePlayer" :direction="directionTwo">
      <div class="block">
        <div v-if="isPlay">
          <video-player
            class="video-player-box vjs-custom-skin"
            ref="videoPlayer"
            :options="playerOptions"
            :playsinline="true"
          ></video-player>
        </div>

        <MarqueeTips
          v-if="close"
          content="若视频无法自动播放，请手动点击播放器左下角的播放按钮即可(20秒后自动关闭本提示)"
          :speed="10"
          style="margin-top: 20px;color: red;"
        ></MarqueeTips>
      </div>
    </el-drawer>
  </div>
</template>

<script>
import axios from "axios";
import { Toast } from "mint-ui";
import MarqueeTips from "vue-marquee-tips";

export default {
  name: "home",
  created() {
    this.getCate();
  },
  data() {
    return {
      isCheck: null,
      search: "",
      movieImg: [
        "https://cdn.92lzx.cn/movieOne.jpeg",
        "https://cdn.92lzx.cn/movieTwo.jpeg",
        "https://cdn.92lzx.cn/movieThere.jpeg",
        "https://cdn.92lzx.cn/movieFour.jpeg"
      ],
      player: false,
      drawer: false,
      direction: "ttb",
      directionTwo: "btt",
      loading: false,
      noData: false,
      pageInfo: {
        id: null,
        page: 1,
        total: 0
      },
      options: [], // 下拉框
      value: "", // 下拉框选中id
      listData: [],
      playerOptions: {
        muted: true,
        // playbackRates: [0.7, 1.0, 1.5, 2.0], //播放速度
        autoplay: true, //如果true,浏览器准备好时开始回放。
        controls: true, //控制条
        preload: "auto", //视频预加载
        muted: false, //默认情况下将会消除任何音频。
        loop: false, //导致视频一结束就重新开始。
        language: "zh-CN",
        aspectRatio: "16:9", // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
        fluid: true, // 当true时，Video.js player将拥有流体大小。换句话说，它将按比例缩放以适应其容器。
        sources: [
          {
            type: "application/x-mpegURL",
            src: "https://www.gentaji.com/20200306/72UI5JjT/index.m3u8"
          }
        ],
        poster:
          "https://cdn.xiaohuwei.cn/71591d4e11e8295f2cf94237016af43e?imageView2/2/w/1920/q/75/format/webp", //你的封面地址
        width: document.documentElement.clientWidth,
        notSupportedMessage: "此视频暂无法播放，请稍后再试", //允许覆盖Video.js无法播放媒体源时显示的默认信息。
        controlBar: {
          timeDivider: true // 当前时间和持续时间的分隔符
        }
      },
      close: true,
      isPlay: false,
      closeTip: 20
    };
  },
  methods: {
    getCate() {
      axios.get("https://api.wdnm.icu/av/getcategory.php").then(res => {
        this.options = [...res.data.rescont];
      });
    },
    checkCate(val) {
      this.search = "";
      this.loading = true;
      this.noData = false;
      this.pageInfo.id = Number(val);
      this.pageInfo.page = 1;
      axios
        .get("https://api.wdnm.icu/av/getlist.php", {
          params: { ...this.pageInfo }
        })
        .then(res => {
          this.listData = [...res.data.rescont.data];
          this.pageInfo.total = res.data.rescont.total;
        })
        .finally(() => {
          this.loading = false;
        });
    },
    searchVideo() {
      if (this.search == "") {
        this.$notify({
          title: "警告",
          message: "请输入要搜索的内容",
          type: "warning",
          position: "bottom-left"
        });
      } else {
        this.loading = true;
        this.value = "";
        this.pageInfo.page = 1;
        let data = {
          page: this.pageInfo.page,
          search: this.search
        };
        axios
          .get("https://api.wdnm.icu/av/search.php", {
            params: { ...data }
          })
          .then(res => {
            if (res.data.rescont.data.length === 0) {
              this.noData = true;
              this.listData = [...res.data.rescont.data];
              this.pageInfo.total = res.data.rescont.total;
            } else {
              this.noData = false;
              this.listData = [...res.data.rescont.data];
              this.pageInfo.total = res.data.rescont.total;
            }
          })
          .finally(() => {
            this.loading = false;
          });
      }
    },
    changePage(val) {
      this.isCheck = null;
      this.pageInfo.page = val;
      axios
        .get("https://api.wdnm.icu/av/getlist.php", {
          params: { ...this.pageInfo }
        })
        .then(res => {
          this.listData = [...res.data.rescont.data];
          this.pageInfo.total = res.data.rescont.total;
        })
        .finally(() => {
          this.loading = false;
        });
    },
    handleSelect(item) {
      this.toPlay(item.id);
    },
    change() {
      this.pageInfo.page = 1;
    },
    toPlay(id, index) {
      this.isCheck = index;
      this.isPlay = true;
      this.player = true;
      axios
        .get("https://api.wdnm.icu/av/getcontent.php", {
          params: {
            id
          }
        })
        .then(res => {
          let src = res.data.rescont.videopath;
          let myPlayer = this.$refs.videoPlayer.player;
          myPlayer.src(src);
          this.closeMarquee();
        })
        .catch(e => {
          this.loading = false;
          Toast("获取资源失败，请点击右上角的刷新按钮再试一次吧");
        });
    },
    closeMarquee() {
      let times = setInterval(() => {
        this.closeTip--;
        if (this.closeTip == 0) {
          this.close = false;
          clearInterval(times);
        }
      }, 1000);
    },
    closePlayer() {
      this.isPlay = false;
    }
  },
  components: {
    MarqueeTips
  }
};
</script>

<style lang="scss" scoped>
.home-container {
  .check {
    background: rgb(243, 186, 94);
    color: #fff;
  }
  /deep/ .el-drawer__header {
    margin-bottom: 10px;
    padding: 10px 10px 0;
  }
  /deep/ .el-drawer__open .el-drawer.ttb {
    height: 70% !important;
  }
  .search {
    margin-top: 15px;
    /deep/ .el-input {
      width: 77%;
    }
  }
  .author {
    position: fixed;
    top: 100px;
    left: 0;
    background: rgb(243, 186, 94);
    color: #fff;
    padding: 5px;
    // font-size: 16px;
  }
  /deep/ .el-select {
    width: 100%;
  }
}
.page {
  margin-bottom: 20px;
  text-align: center;
}
.list {
  padding: 20px;
  box-sizing: border-box;
  ul {
    li {
      border: 1px solid #ccc;
      border-radius: 10px;
      padding: 20px;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      margin-bottom: 20px;
      img {
        width: 80%;
      }
      p {
        font-size: 18px;
        .titles {
          color: rgb(255, 86, 34);
        }
      }
    }
  }
}
</style>