<template>
  <v-container>
    <div>
    <v-sheet width="300" class="mx-auto">
      <v-form @submit.prevent>
        <v-text-field
          v-model="UserId"
         
          label="UserId"
        ></v-text-field>
        <v-text-field
          v-model="CourseId"
          
          label="CourseId"
        ></v-text-field>
        <v-btn type="submit" block class="mt-2" @click="addCourse()">add course to user</v-btn>
      </v-form>
    </v-sheet>
    <div>
      <h2>List of User Course</h2>
      <ul v-if="courselist.length > 0">
        <li v-for="course in courselist" :key="course.id">
          
          <v-card>{{ course }} </v-card>
        </li>
      </ul>
      <p v-else>No course found for this user.</p>
      <v-btn @click="getCourse()">getcourse</v-btn>
    </div>
  </div>
  </v-container>
  </template>

<script>
import constant from '@/utils/constant';
import axios from 'axios';

export default {
  data: () => ({
    CourseId: '',
    UserId:'',
    courselist:[],
    
  }),

  methods:{
    async addCourse(){

        try {
            axios.post(constant.localDomain+'/addCourse',null,{
                params:{
                    CouserId:this.CourseId,
                    UserId: this.UserId,
                }
            }).then((Response)=>{
                console.log('success? :',Response.data)
            })
            
        } catch (error) {
          if (error.status==400) {
            console.log('This course is already exist in user')
          }
            console.log('Error : ',error)
        }
   
    },
    async getCourse(){
      

      await axios.get(constant.localDomain+'/userCourseList/'+1).then((Response)=>{
        this.courselist=Response.data;
      })
    }
  }
}
</script>