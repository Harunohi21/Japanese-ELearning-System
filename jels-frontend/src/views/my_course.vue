<template>
  <div class="scrollable-container">
    <v-row>

      <!-- Course List -->
      <v-col cols="10">

        <v-row>
          <v-col cols="10">
            <!--
          <v-card
            class="mx-auto"
            color="grey-lighten-3"
            max-width="800">
            <v-card-text>
              <v-text-field
                :loading="loading"
                density="compact"
                variant="solo"
                label="Search templates"
                append-inner-icon="mdi-magnify"
                single-line
                hide-details
                @click:append-inner="onClickTitle"
              ></v-text-field>
            </v-card-text>
          </v-card>
        -->
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="3" v-for="(mycourse, index) in mycourseList" :key="index">
            <v-card @click="goToCourseVideo (mycourse)" height="350">
              <v-card-text>
                <v-img
                  :src="localDomain +'/'+ mycourse.imagePath"
                  max-height="200"
                  cover
                ></v-img><br/>
                <div class="text-h6">{{ mycourse.title }}</div>
                <div class="text-body-1">{{ mycourse.refBook }} </div>
                <v-btn color="#FFA000"  @click="goToCourseVideo (mycourse)"> View </v-btn>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
        
      </v-col>
    </v-row>
  </div>
</template>
 <!--
<style>
.scrollable-container {
  /* Add additional styles to the container */
  height: 300px;
  overflow-y: scroll;
}
</style>-->
<script>
import utils from "../utils/utils";

export default {
  name: "MyCourse",

  components: {},

  data() {
    return {
      loginUser:{},
      localDomain: utils.constant.localDomain,
      //imagePath: utils.constant.imagePath,
      mycourseList: [],
     
      
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
    await this.fetchMyCourses();
  },

  methods: {
    async fetchMyCourses() {
      const resp = await utils.http.get("/enroll/users/"+this.loginUser.id+"/courses");
      if (resp && resp.status === 200) {
        const data = await resp.json();
        if (data) {
          this.mycourseList = data;
        }
      }
    },

    goToCourseVideo(course) {
      this.$router.push({
        path: "/course/" + course.id+"/course_videos",
      });
    },

    
  },
};
</script>
