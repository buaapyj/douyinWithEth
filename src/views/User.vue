<template>
  <div>
    <b-row align-h="center">
      <b-col cols="3">
        <UserCard :user="userData"></UserCard>
      </b-col>
    </b-row>
    <b-row>
      <b-col sm="6" md="4" lg="3" xl="2" v-for="video in videoData" :key="video.url">
        <VideoCard :video="video"></VideoCard>
      </b-col>
    </b-row>
  </div>
</template>

<script>
  import UserCard from "../components/UserCard";
  import VideoCard from "../components/VideoCard";
  export default {
    name: "User",
    components: {VideoCard, UserCard},
    data(){
      return {
        userAddress :'',
        userData: {
          userAddress: '',
          userHash: '',
          userVideoNums: 0,
          userAlbumNum: 0,
          userGratuityCount: 0,
          userGratuitySum: 0
        },
        videoData: [],
      }
    },
    methods:{
      async init(){
        this.userAddress = this.$route.params.address;
        const dikTok = this.$store.state.dikTok;
        this.userData = await dikTok.methods.getUserInfo(this.userAddress).call().then((res)=>{return res});
        this.userData['userAddress'] = this.userAddress;
        this.videoData = [];
        for (let i = 0; i < this.userData.userVideoNums && i < 6; i++) {
          const video = await dikTok.methods.getUserVideo(this.userAddress, i).call().then((res)=>{return res});
          video['author'] = this.userAddress;
          this.videoData.push(video);
        }
      }
    },
    created() {
      this.init()
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