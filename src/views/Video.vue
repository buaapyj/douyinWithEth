<template>
  <div>
    <b-row>
      <b-col cols="9">
        <VideoPlayer :video="videoData" v-on:playend="playend"></VideoPlayer>
      </b-col>
      <b-col cols="3">
        <UserCard :user="userData"></UserCard>
        <CommentCard :comment="comment" v-for="comment in CommentData" :key="comment.userHash"></CommentCard>
        <FunctionCard :videoid="videoId"></FunctionCard>
      </b-col>
    </b-row>
    <b-row>
      <b-pagination-nav :link-gen="getNum" @change="gotoplayer" :number-of-pages="MaxPage" use-router></b-pagination-nav>
    </b-row>
    <b-row>
      <b-col sm="6" md="4" lg="3" xl="2" v-for="v in videolist" :key="v.videoHash">
        <VideoCard :video="v"></VideoCard>
      </b-col>
    </b-row>
  </div>
</template>

<script>
  import VideoPlayer from "../components/VideoPlayer";
  import UserCard from "../components/UserCard";
  import CommentCard from "../components/CommentCard";
  import FunctionCard from "../components/FunctionCard";
  import VideoCard from "../components/VideoCard";
  export default {
    name: "Video",
    components: {VideoCard, FunctionCard, CommentCard, UserCard, VideoPlayer},
    data(){
      return {
        videoId :0,
        videoData: {
          videoID: 0,
          videoHash: '',
          duration: 0,
          timestamp: 0,
          author: '',
          commentsNum: 0,
          vlableNum: 0,
          gratuityNum: 0,
          gratuitySum: 0,
        },
        userData: {
          userAddress: '',
          userHash: '',
          userVideoNums: 0,
          userAlbumNum: 0,
          userGratuityCount: 0,
          userGratuitySum: 0
        },
        CommentData: [],
        MaxPage:0,
        MaxVideo:0,
        videolist:[],
      }
    },
    methods:{
      async init(){
        this.videoId = this.$route.params.id;
        const wait = setInterval(()=>{
            if(this.$store.state.dikTok!=null){
                window.clearInterval(wait);
                this.initplayer();
            }
        },1000)
      },
      async initplayer(){
          const dikTok = this.$store.state.dikTok;
          this.videoData = await dikTok.methods.getVideo(this.videoId).call().then((res)=>{return res});
          this.videoData['videoID'] = this.videoId;

          this.userData = await dikTok.methods.getUserInfo(this.videoData.author).call().then((res)=>{return res});
          this.userData['userAddress'] = this.videoData.author;

          this.CommentData = [];
          for (let i = 0; i < this.videoData.commentsNum && i < 6; i++) {
              const comment = await dikTok.methods.getVideoComment(this.videoId, i).call().then((res)=>{return res});
              this.CommentData.push(comment);
          }
      },
      getNum(pageNum){
        return '/video/'+(pageNum-1)*10
      },
      async initvideolist(){
          const dikTok = this.$store.state.dikTok;
          let vid = await dikTok.methods.videoNum().call().then((res)=>{return res});
          this.MaxVideo = vid;
          this.MaxPage = Math.ceil(vid/10);
          await this.gotoplayer(this.MaxPage);
      },
      async gotoplayer(pageNum){
        const dikTok = this.$store.state.dikTok;
        let start = (pageNum - 1) * 10;
        let end = pageNum * 10;
        end = end > this.MaxVideo ? this.MaxVideo - 1 : end;
        this.videolist = [];
        for (let i = start; i <= end; i++) {
          const v = await dikTok.methods.getVideo(i).call().then((res)=>{return res});
          v['videoId'] = i;
          this.videolist.push(v);
        }
      },
        playend(){
          if(this.videoId>0){
              this.$router.push({path:'/video/'+(+this.videoId-1)})
          }
        }
    },
    created() {
      const initvl = setInterval(()=>{
          if(this.$store.state.dikTok!=null){
              this.initvideolist();
              clearInterval(initvl);
          }
      },2000)
      this.init();
    },
    watch:{
      $route(to,from){
        this.init();
      },
    }
  }
</script>

<style scoped>

</style>