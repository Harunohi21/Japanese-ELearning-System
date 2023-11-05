<template>
  <div>
    <v-row ><v-col cols="12" ></v-col> </v-row>
    <v-row>
      <v-col cols="2">
        </v-col>
      <!-- Course Video List -->
      <v-col cols="8">
        <v-row>
          <v-col >
            <div >
            <v-card class="card-container" height="600">
              <v-card-text >
               <!-- <div class="text-h6">Day</div>-->
        
                <video
                  width="98%"
                  :src="localDomain + '/'+currentVideo.path"
                  controls
                ></video>
                
               <!-- <div class="text-h6 text-center">{{ courseVideo.title }}</div>-->
               <div class="text-h6 ">{{ currentVideo.title }}</div>
              
                <v-row justify="center">
                  <v-col cols="2" ></v-col>
                  <v-col cols="3" ><button @click="navigateToPreviousVideo" ><svg-icon type="mdi" :path="pathLeft" style="width: 60px; height: 50px; fill: #ffd343;"></svg-icon> </button></v-col>
                  <v-col cols="3" > <button @click="navigateToNextVideo"><svg-icon type="mdi" :path="pathRight" style="width: 60px; height: 50px;"></svg-icon></button>
                  <v-col cols="4" ></v-col>
                  </v-col></v-row>
                
              </v-card-text>
            </v-card></div>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>

<style scoped>
.card-container {
  justify-content: center;
  align-items: center;
  align-content: center;
  /*height: 100vh; 
   display: flex;*/
}

</style>
<script>
import utils from "../utils/utils";
//import axios from 'axios';
import SvgIcon from '@jamescoyle/vue-icon'
import { mdiMenuLeft } from '@mdi/js';
import { mdiMenuRight } from '@mdi/js';



export default {
  
  name: "courseVideoDetail",
  components: {
    SvgIcon
  },

  data() {
    return {
      loginUser:{},
      localDomain: utils.constant.localDomain,
      course_video_id: this.$route.params.id,
      course_id:this.$route.params.course_id,

      videoList: [],
      currentVideoIndex: 0,
      currentVideo: null,
      pathLeft: mdiMenuLeft,
      pathRight: mdiMenuRight,
    };
  },

  async created() {
    // LoginUser from Vuex
    this.loginUser = this.$store.state.loginUser;
    this.$store.watch(
      () => {
        return this.$store.state.loginUser;
      },
      (newVal, oldVal) => {
        this.loginUser = newVal;
      },
      {
        deep: true,
      }
    );
    this.fetchVideoList();
   
  },
  
  //Get CourseVideo By Course ID
  methods: {


    async fetchVideoList() {
      const resp = await utils.http.get("/course/"+this.course_id+"/course_videos");
      if (resp && resp.status === 200) {
        const data = await resp.json();
        if (data) {
          console.log("data",data)
          this.videoList = data;

          const resp = await utils.http.get("/course_video/"+this.course_video_id);
            if (resp && resp.status === 200) {
              const data = await resp.json();
              if (data) {
                this.currentVideo = data;
              }
            }
        }//if
      }//if          
    },

    async navigateToPreviousVideo() {
      this.course_video_id--;
      await this.fetchVideoList();
    },
    async navigateToNextVideo() {
      this.course_video_id++;
      await this.fetchVideoList();
    },




  },//Method
};//Export
</script>
