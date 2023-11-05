<template>
  <div>
 <div v-if="!paymentStatus">
      

  <v-card
    class="mx-auto"
    max-width="500"
    tile
  >
    <v-list rounded>
      <v-subheader>Cards</v-subheader>
      <v-list-item-group
        v-model="selectedCard"
        color="primary"
      >
        <v-list-item
          v-for="card  in cards"
          :key="card.id"
        >
          
          <v-list-item-content>
            <v-list-item-title v-text="card.card.last4"></v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list-item-group>
    </v-list>
  </v-card>
    
    
    <h2>My Cards</h2>
    <v-card v-for="card in cards" :key="card.id" @click="selectCard(card)">
      <v-card-text>
        <div>
          <strong>{{ card.card.brand }}</strong>
        </div>

        <div>
          Ending in {{ card.card.last4 }}
        </div>
        
      </v-card-text>
    </v-card>
    <v-btn v-if="selectedCard" @click="handlePayment">Use this Card</v-btn>
    <v-btn  @click="payWithNewCard">Pay With New Card</v-btn>
  </div>
  <div v-if="paymentStatus">
    You Already paid ,
    <v-btn @click="pushToCourse">Go To Course</v-btn>
  </div>
  


</div>
</template>
  
  <script>
  import constant from '@/utils/constant';
import http from '@/utils/http';
import axios from 'axios';
  
  export default {
    data() {
      return {
        paymentStatus:false,
        loginUser:{},
         
        cards: [],
        selectedCard:null,
        course:{},
        course_id: this.$route.params.id,
      };
    },
    async mounted() {
      await this.fetchCustomerCards();
      await this.fetchCourse();
      await this.getPaymentStatus();
      await this.pushToCourse();
       
    },
     created(){
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
     

    },
    methods: {

      async getPaymentStatus() {
      const resp = await  axios.get(constant.localDomain +"/enroll/users/" +this.loginUser.id+"/courses");
      console.log('Response status', resp.status);
      console.log('Response Data', resp.data);

      if (resp && resp.status === 200) {
        const data = resp.data; // Access response data directly
        if (data) {
          this.user_courses = data; // Assign response data to user_courses
        }
      }
      
      for (let course of this.user_courses) {
        console.log('this.course_id  ', this.course_id  );

        console.log('Course Id', course.id );
          if(parseInt(this.course_id) === course.id ) {
            console.log('True Status');
            this.paymentStatus=true;
            break;
          }
          else{
            console.log('error');
          }
      }
      console.log("paid ? ",this.paymentStatus);

    },
      fetchCustomerCards() {
        
        axios.get(constant.localDomain+'/cardList',{
           params:{
          userId:this.loginUser.id,
          
        }
        })
          .then(response => {
            this.cards = response.data;
          })
          .catch(error => {
            console.error('Error fetching customer cards:', error);
          });

          
      },
      async selectCard(card) {
      // Set the selected card in the 'selectedCard' data property
      this.selectedCard = card;
    },

    async fetchCourse() {
      console.log("fetching course fee");
      const resp = await http.get("/course/" + this.course_id);
      if (resp && resp.status === 200) {
        const data = await resp.json();
        if (data) {
          this.course = data;
        }

      }
      console.log("course ", this.course);
    },

    async handlePayment(){
      console.log("handleing payment ",this.selectedCard.id,this.course.fee);
      console.log("user id ", this.loginUser);
      const resp= await axios.post(constant.localDomain+'/payment/charge',null,{
        params:{
        amount:this.course.fee*100,
        user_id:this.loginUser.id,
        paymentMethodId:this.selectedCard.id,
        }
      })
      if(resp.status===200){
        console.log("ok");

         //go to BackEnd 
         
          const resp = await  axios.post(constant.localDomain +"/enroll/userId/"+this.loginUser.id+"/courseId/"+this.course_id);
           console.log("enroll ok?",resp.data);//String Response
            if (resp && resp.status === 200) {
              //console.log('Enroll Status:', resp.status);
              //Frontend Route
             this.$router.push({ path: "/course_detail/" + this.course_id});
              
            }
           else { console.log("error add course");} 

        
      }else{
        console.log("error payment");
      }
      
    },
    payWithNewCard(){
      this.$router.push({
        path: "/payment/course/"+this.course_id,
      });
    },
    pushToCourse(){
      if(this.paymentStatus==true){
        this.$router.go(-1);
      }
    } 

    },
  };
  </script>
  