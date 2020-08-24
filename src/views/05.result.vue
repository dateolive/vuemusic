<template>
  <div class="result-container">
    <div class="title-wrap">
      <!-- 标题 -->
      <h2 class="title">{{ $route.query.q }}</h2>
      <span class="sub-title">找到 {{ count }} 个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr
              v-for="(item, index) in songList"
              :key="index"
              class="el-table__row"
              @dblclick="playMusic(item.id)"
            >
              <td>{{ index+1 }}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <!-- 歌名 -->
                    <span>{{ item.name }}</span>
                    <!-- mv的图标 -->
                    <span v-if="item.mvid != 0" class="iconfont icon-mv"></span>
                  </div>
                  <!-- 二级标题 -->
                  <span v-if="item.alias.length != 0">{{ item.alias[0] }}</span>
                </div>
              </td>
              <td>{{ item.artists[0].name }}</td>
              <td>{{ item.album.name }}</td>
              <td>{{ item.duration }}</td>
            </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div
            v-for="(item, index) in playList"
            :key="index"
            class="item"
            @click="toPlaylist(item.id)"
          >
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div v-for="(item, index) in mv" :key="index" class="item" @click="toMV(item.id)">
            <div class="img-wrap">
              <!-- 封面 -->
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <!-- 播放次数 -->
                <div class="num">{{ item.playCount }}</div>
              </div>
              <!-- 持续时间 -->
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <!-- mv名 -->
              <div class="name">{{ item.name }}</div>
              <!-- 歌手名 -->
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
<!-- 分页器 -->
      <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="10"
        :limit="limit"
      >
      </el-pagination>
  </div>
</template>

<script>
  // 导入 aixos
  import axios from 'axios'
  export default {
    name: 'result',
    data() {
      return {
        // tab切换时 会改变的数据
        activeIndex: 'songs',
        total: 0,
        page:1,
        limit:10,
        // 保存 查询歌曲
        songList: [],
        // 保存歌单的字段
        playList: [],
        // 保存mv的字段
        mv: [],
        // 搜索结果的总数
        count: 0,
        type:1
      }
    },
    // 生命周期钩子 回调函数
    created() {
      this.search()
    },
    // 侦听器
    watch: {
      activeIndex() {
        // songs 歌曲
        // lists 歌单
        // mv    mv
        this.page=1

        
        switch (this.activeIndex) {
          case 'songs':
            this.type = 1
            break
          case 'lists':
            this.type = 1000
            break
          case 'mv':
            this.type = 1004
            break

          default:
            break
        }

        axios({
          url: 'https://autumnfish.cn/search',
          method: 'get',
          params: {
            keywords: this.$route.query.q,
           
            // 获取的数据量
            limit:this.limit, // limit: limit
            //分页
            offset:(this.page-1)*this.limit,
             type:this.type // type:type,
          }
        }).then(res => {
          //console.log(res)
          // 获取歌曲
          if (this.type == 1) {
            // 歌曲
            this.songList = res.data.result.songs
            this.count=res.data.result.songCount
            this.total=res.data.result.songCount
            // 计算歌曲时间
           for(let i=0;i<this.songList.length;i++){
                let duration = this.songList[i].duration;  
                let min = parseInt(duration/1000/60);
                if(min<10){
                      min='0'+min
                }
                let sec = parseInt(duration/1000%60)
                    if(sec<10){
                      sec='0'+sec
                    }
                    //console.log(min+" "+sec)
                    this.songList[i].duration=`${min}:${sec}`
          }
          }
          // 获取 歌单
          else if (this.type == 1000) {
            // 歌单的逻辑
            this.playList = res.data.result.playlists
            this.total=res.data.result.playlistCount
            // 处理 播放次数
            for (let i = 0; i < this.playList.length; i++) {
              if (this.playList[i].playCount > 100000) {
                this.playList[i].playCount =parseInt(this.playList[i].playCount / 10000) + '万'
              }
            }
          } else {
            // 保存mv数据
            this.mv = res.data.result.mvs
            // 总数
            this.total=res.data.result.mvCount
            // 处理数据
            for(let i=0;i<this.mv.length;i++){
                  if(this.mv[i].playCount>100000){
                    this.mv[i].playCount=parseInt(this.mvt[i].playCount/10000)+'万'
                  }
                }
          }
        })
      }
    },
    // 方法
    methods: {
      search(){
        axios({
        url: 'https://autumnfish.cn/search',
        method: 'get',
        params: {
          keywords: this.$route.query.q,
          limit: this.limit,
          
          offset:(this.page-1)*this.limit,
          // 获取的数据量
          type: 1
          
        }
      }).then(res => {
        //console.log(res)
        //判断搜索内容是否报错  例如搜索了不该搜索的内部
         if(res.data.hasOwnProperty('msg')){
            this.$message({
              message: '搜索内容异常！',
              type: 'warning'
            })
          }else{
          this.songList=res.data.result.songs;
          this.count=res.data.result.songCount
          this.total=res.data.result.songCount
          for(let i=0;i<this.songList.length;i++){
                  let duration = this.songList[i].duration;  
                  let min = parseInt(duration/1000/60);
                  if(min<10){
                    min='0'+min
                  }
                  let sec = parseInt(duration/1000%60)
                  if(sec<10){
                    sec='0'+sec
                  }
                  //console.log(min+" "+sec)
                  this.songList[i].duration=`${min}:${sec}`
                }
            }
        },err=>{
          console.log(err)
        })
      },
      // 去mv详情页
      toMV(id){
        this.$router.push(`/mv?q=${id}`)
      },
      // 去歌单详情页
      toPlaylist(id){
        // 跳转并携带数据即可
        this.$router.push(`/playlist?q=${id}`)
      },
      playMusic(id) {
        axios({
          url: 'https://autumnfish.cn/song/url',
          method: 'get',
          params: {
            id // id:id
          }
        }).then(res => {
          // console.log(res)
          let url = res.data.data[0].url
          // console.log(this.$parent.musicUrl)
          // 设置给父组件的 播放地址
          this.$parent.musicUrl = url
        })
      },
      handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page=val;
         axios({
          url:' https://autumnfish.cn/search',
          method:'get',
          params:{
            keywords:this.$route.query.q,
            limit:this.limit,
            offset:(this.page-1)*this.limit,
            type:this.type
          }
        }).then(res => {
          console.log(res);
          if(this.type==1){
              this.songList=res.data.result.songs;
                this.count=res.data.result.songCount
                this.total=res.data.result.songCount
                for(let i=0;i<this.songList.length;i++){
                    let duration = this.songList[i].duration;  
                    let min = parseInt(duration/1000/60);
                    if(min<10){
                      min='0'+min
                    }
                    let sec = parseInt(duration/1000%60)
                    if(sec<10){
                      sec='0'+sec
                    }
                    //console.log(min+" "+sec)
                    this.songList[i].duration=`${min}:${sec}`
                  }
          }else if(this.type==1000){
                this.playList=res.data.result.playlists;
                this.total=res.data.result.playlistCount
                for(let i=0;i<this.playList.length;i++){
                  if(this.playList[i].playCount>100000){
                    this.playList[i].playCount=parseInt(this.playList[i].playCount/10000)+'万'
                  }
                }
          }else if(this.type==1004){
                this.mv=res.data.result.mvs;
                this.total=res.data.result.mvCount
                for(let i=0;i<this.mv.length;i++){
                  if(this.mv[i].playCount>100000){
                    this.mv[i].playCount=parseInt(this.mv[i].playCount/10000)+'万'
                  }
                }
          }

        })
    }
      
      
    }
  }
</script>

<style></style>
