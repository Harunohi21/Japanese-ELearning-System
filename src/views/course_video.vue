<template>
  <div>
    
    <v-row >
      <v-col cols="2" justify="center">  
        <v-card color="#ffd343" class="mx-auto " width="300" height="800" >
          <v-row ><v-col cols="12" class="mt-7"></v-col> </v-row>
          <div class="text-h4 text-center" >{{course.title}}</div>
          <v-row ><v-col cols="12" class="mt-7"></v-col> </v-row>
          <div>
          <!--<a href="#" @click="downloadBook">Download Book</a>-->
          <center><v-btn class="custom-button" color="#000000" style="color: #ffd343;" @click="downloadBook()">Download Book</v-btn></center>
         </div>
  
        </v-card>
      </v-col>
      
      <!-- Course Video List -->
      <v-col cols="10">
        <v-row ><v-col cols="12" class="mt-5"></v-col> </v-row>
        <v-row>
          <v-col cols="3" v-for="(courseVideo, index) in courseVideoList" :key="index">
            <v-card  @click=" goToCourseVideoDetail(courseVideo)" height="220">
              <v-card-text>
               <!-- <div class="text-h6">Day</div>-->
                <video ref="videoPlayer" 
                  class="mb-2"
                  width="100%"
                  :src="localDomain + '/'+courseVideo.path"
                > </video>
                <div class="text-h6 text-center">{{ courseVideo.title }}</div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>
<style scoped>

.custom-button {
  border: 10px solid #eee9e9;
  border-radius: 16px;
  
}
</style>

<script>
import utils from "../utils/utils";

export default {
  name: "courseVideo",

  components: {},

  data() {
    return {
      loginUser:{},
      localDomain: utils.constant.localDomain,
      course_id: this.$route.params.id,
      course: [],
      courseVideoList: [],
      courseLevelList: [],
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
    await this.fetchCourse();
    await this.fetchCourseVideo();
  },
  
  //Get CourseVideo By Course ID
  methods: {

    async fetchCourse() {
      const resp = await utils.http.get("/course/" + this.course_id);
      if (resp && resp.status === 200) {
        const data = await resp.json();
        if (data) {
          this.course = data;
        }
      }
    },

    async fetchCourseVideo() {
      const resp = await utils.http.get("/course/"+this.course_id+"/course_videos");
      if (resp && resp.status === 200) {
        const data = await resp.json();
        if (data) {
          this.courseVideoList = data;
        }
      }
    },

    downloadBook() {
    
    console.log(this.course);

    const link = document.createElement('a');
    const pdfUrl = this.localDomain + '/' + this.course.pdfPath; // Concatenate the URL parts correctly
    link.href = pdfUrl;
    link.download = 'book.pdf'; // Specify the desired file name for the downloaded book
    link.click(); 

    //for a href
    //const pdfUrl = this.localDomain + '/' + this.course.pdfPath; // Concatenate the URL parts correctly
    //window.location.href = pdfUrl;

      //const bookPath =this.localDomain + '/'+this.course.pdfPath; 
      //console.log(this.localDomain + '/'+this.course.pdfPath);
      //const link = document.createElement('a');
      //link.href = window.URL.createObjectURL(new Blob([this.localDomain + '/'+this.course.pdfPath], { type: 'application/pdf' }));
      //console.log("link href",link.href);
      //link.download = 'book.pdf'; // Specify the desired file name for the downloaded book
      //link.click();

  
    },



    goToCourseVideoDetail(course_video) {
      this.$router.push({
        path: "/course/"+this.course_id+"/course_video_detail/" + course_video.id,
      });
    },


    


  },//Method
};//Export
</script>
