<template>
  <!-- <div class="course_detail background-image" >-->

  <div class="course_detail">

    <v-row><v-col cols="12" class="mt-20"></v-col> </v-row>
    <v-row><v-col cols="12" class="mt-12"></v-col></v-row>

    <v-row justify="center" class="ma-0">

      <v-col cols="2" class=" ma-0 pa-0"> </v-col>

      <!-- Image -->
            <v-img class="ma-0" height="480" :src="localDomain + '/' + course.imagePath" contain>
            </v-img>


            <!-- Course Info -->
            <v-col cols="4">
              <div class="text-h3 "> {{ course.title }} </div><br/>
              <div class="text-body-1 ml-2 my-2">Reference Book:{{ course.refBook }}</div>
              <div class="text-body-1 ml-2 my-2">Total Video: {{ videoCount }} </div>
              <div class="text-body-1 ml-2 my-2">Fee:{{ course.fee }} MMK</div>
              <div class="text-body-1 ml-2 my-2">Description: {{ course.description }}</div>
              <!-- <v-btn @click="goToCourseVideo(course)"> View </v-btn>-->
              <v-spacer></v-spacer>
              <!--  <v-btn @click="goToPayment()"> {{ buttonText }} </v-btn> -->
              <br/>
              <v-btn color="#ffd343" v-if="!paymentStatus" @click="goToPayment(course)"> Enroll </v-btn>
              <v-btn color="#ffd343" v-if="paymentStatus" @click="goToCourseVideo(course)"> View </v-btn>
            </v-col>

      <v-col cols="1"></v-col>
    </v-row>

  </div>
</template>
<style scoped>
.background-image {
  /* Set the background image properties */
  background-image: url('../assets/AllBack2.jpg');
  background-size: cover;
  background-position: center;
  /* Add any other styles you need for the component */
}

.course-detail {
  justify-content: center;
  align-items: center;
  align-content: center;
  /*height: 100vh; 
   display: flex;*/
}

.container {
  margin-left: 10vw !important;
  margin-right: 10vw !important;
}
</style>
<script>
import utils from "../utils/utils";
import axios from 'axios';
import constant from '@/utils/constant';

export default {
  name: "course_detail",

  data()
  {
    return {
      loginUser: {},
      localDomain: utils.constant.localDomain,
      course_id: this.$route.params.id,

      course: [],
      user_courses: [],
      paymentStatus: false,//this.loginUser.paymentStatus
      buttonText: "",
      userCard: [],
      courseVideoList: [],
    };
  },
  computed: {
    videoCount()
    {
      return this.courseVideoList.length;
    },
  },


  async created()
  {
    // LoginUser from Vuex
    this.loginUser = this.$store.state.loginUser;
    this.$store.watch(
      () =>
      {
        return this.$store.state.loginUser;
      },
      ( newVal, oldVal ) =>
      {
        this.loginUser = newVal;
      },
      {
        deep: true,
      }
    );

    await this.fetchCourse();
    await this.fetchCourseVideo();

    //await this.fetchButtonText();
    await this.getPaymentStatus();

    await this.getCards();
  },

  methods: {

    async fetchCourseVideo()
    {
      const resp = await utils.http.get( "/course/" + this.course_id + "/course_videos" );
      if ( resp && resp.status === 200 ) {
        const data = await resp.json();
        if ( data ) {
          this.courseVideoList = data;
        }
      }
    },

    async getPaymentStatus()
    {
      const resp = await axios.get( constant.localDomain + "/enroll/users/" + this.loginUser.id + "/courses" );
      console.log( 'Response status', resp.status );
      console.log( 'Response Data', resp.data );

      //const resp = await utils.http.get("/enroll/users/" + this.user_id+"/courses");

      //if (resp && resp.status === 200) {
      // const data = await resp.json();
      //if (data) {
      // this.user_courses = data;
      //}}

      if ( resp && resp.status === 200 ) {
        const data = resp.data; // Access response data directly
        if ( data ) {
          this.user_courses = data; // Assign response data to user_courses
        }
      }

      for ( let course of this.user_courses ) {
        console.log( 'this.course_id  ', this.course_id );
        //console.log('Course ', course );
        console.log( 'Course Id', course.id );

        // if (typeof this.course_id === 'number')
        //{console.log('this.course_id is number '  );}

        // if (typeof course.id === 'number')
        //{console.log('course.id  is number' );}

        if ( parseInt( this.course_id ) === course.id ) {
          console.log( 'True Status' );
          this.paymentStatus = true;
          break;
        }
        else {
          console.log( 'error' );
        }
      }

    },

    // async fetchButtonText() {
    // if (this.paymentStatus === false) {
    // this.buttonText= "Enroll";       
    //  }
    // else{
    // this.buttonText= "View";
    // }
    // },

    async fetchCourse()
    {
      const resp = await utils.http.get( "/course/" + this.course_id );
      if ( resp && resp.status === 200 ) {
        const data = await resp.json();
        if ( data ) {
          this.course = data;
        }
      }
      console.log( 'image path ', this.course.imagePath );
    },

    goToCourseVideo( course )
    {
      this.$router.push( {
        path: "/course/" + course.id + "/course_videos",
      } );
    },


    goToPayment( course )
    {

      // this.$router.push({
      //   path: "/payment/course/"+course.id,
      // });
      console.log( "total cards ", this.userCard.length );
      if ( this.userCard.length > 0 ) {
        this.$router.push( {
          path: "/charge/course/" + course.id,
        } )
      } else {
        this.$router.push( {
          path: "/payment/course/" + course.id,
        } );
      }
    },
    async getCards()
    {
      const resp = await axios.get( constant.localDomain + '/cardList', {
        params: {
          userId: this.loginUser.id,

        }
      } )
      if ( resp.status == 200 ) {
        console.log( "user has card " );
        this.userCard = resp.data;
      } else if ( resp.status == 404 ) {
        console.log( "user has no card" );
      }

    },

  },
};
</script>
