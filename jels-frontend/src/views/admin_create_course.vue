<template>

    <div>
      <v-row><v-col cols="12" class="mt-12"></v-col></v-row>
      <v-row>
        <!-- Sidebar -->
        <v-col cols="3">
  
        <admin_sidebar></admin_sidebar>
      </v-col>
      <v-col cols="8">
      
    <v-sheet class="mx-auto"  >
      
      <v-form   ref="myform" fast-fail @submit.prevent  >
        <v-text-field
          v-model="title"
          outlined
          dense
          label="Course Title"
          :rules="titleRules"
        ></v-text-field>

        <v-select 
        v-model="level"
        
         dense
        outlined
        item-text="name"
        item-value="id"
        :items="levelList"
        label="Level"
        
        required
        
      ></v-select>

      <v-text-field
          v-model="refBook"
          outlined
          dense
          label="Reference Book"
          
        ></v-text-field>
  
        <v-textarea
        counter
        outlined
        label="Course Description"
        :rules="descripitonRules"
        v-model="descripton"
      ></v-textarea>

      <v-text-field
          v-model="fee"
          outlined
          dense
          label="Course Fee"
          :rules="feeRules"
        ></v-text-field>

    <v-file-input 
      show-size
      outlined
      dense
      prepend-icon=''
      append-icon="mdi-image"
        v-model="imageFile"
      accept=".jpg, .png"
      label="Image"
      @change="onChangeImage"
      
    ></v-file-input>
    <v-img v-if="imagePreviewPath != null"
    :src="imagePreviewPath"
    width="200"
    height="150"
    contain
    ></v-img>

    <v-file-input 
      show-size
      
      dense
      
        
      accept=".pdf"
      label="PDF"
      v-model="pdfFile"
      
    ></v-file-input>

    <v-file-input
      chips
      multiple
      accept=".mp4, .avi"
      
      label="course videos"
      
      v-model="videos"    
    >
  
  </v-file-input>
    
        <v-btn  block class="mt-2" @click="submit" :disabled="!iscomplete">Create Course</v-btn>
        <v-alert type="success"
    v-model="successAlert"
    dismissible
    >
        course created !
      </v-alert>
    <v-alert class="mt-3" v-show="errorAlert" dense type="error">
              Create course Failed !
            </v-alert>
      </v-form>
    </v-sheet>
  </v-col>
  </v-row>
  </div>
  </template>
  
  <script>
  
  import admin_sidebar from '../components/admin_sidebar.vue'
  import constant from '@/utils/constant'
  import http from '@/utils/http'
  import axios from 'axios'
  
    export default {
  
      name:'admin_course_create',
      components:{admin_sidebar},
      data: () => ({

        loginUser:{},
          imageFile: null,
          pdfFile:null,
          title: "",
          titleRules: [
              value => {
                  if (value?.length > 3)
                      return true;
                  return "Title  must be at least 3 characters.";
              },
          ],
          level:1,
          levelList: [],
          descripton: "",
          descripitonRules: [
              value => {
                  if (value?.length > 10) {
                      if (value?.length > 500)
                          return "not more than 500";
                      return true;
                  }
                  return "description must have at least 10";
              }
          ],

          refBook:"",
          fee:null,
          feeRules:[
              (v) => !!v || 'Required',
              (v) =>
                (v && v > 0 && v <= 999999) ||
                'fee must be between 0 and 999999 MMK',
            ],

          videos:[],
         
          imagePreviewPath:null,
          errorAlert: false,
          successAlert: false,
          createdCourse:null,
          
      }),
        
      computed: {
          iscomplete() {
              return this.title && this.descripton && this.imageFile != null && this.level != null
              && this.fee && this.pdfFile != null && this.videos.length>0
              ;
          },
      },
      async created(){
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
        
          await this.fetchLevel();
                },    

      methods: {
          async fetchLevel() {
            console.log("fetching levels");
              await axios.get(constant.localDomain+'/level').then((Response)=>{
                this.levelList=Response.data;
                console.log(this.levelList);
              })
          },
          async submit() {
              let imagePath = null;
              let pdfPath=null;
              const resp = await http.postMedia("/admin/file/create", this.imageFile);
              if (resp.status === 200) {
                  imagePath = await resp.text();
                  console.log(imagePath);
              }
              else {
                  this.errorAlert = true;
              }
              const respPdf = await http.postMedia("/admin/file/create", this.pdfFile);
              if (resp.status === 200) {
                  pdfPath = await respPdf.text();
                  console.log(pdfPath);
              }
              else {
                  this.errorAlert = true;
              }

              
              
             

                  const respCourse= await axios.post(constant.localDomain+ "/admin/course/create ", {
                      title: this.title,
                      level: {id: this.level},
                      refBook:this.refBook,
                      description:this.descripton,
                      fee:this.fee,
                      imagePath: imagePath,
                      pdfPath:pdfPath
                      
                  });
                  if(respCourse && respCourse.status==200){
                    console.log("created : ", respCourse);
                    this.createdCourse=respCourse.data;

                   for (let index = 0; index < this.videos.length; index++) {
                    console.log('current video :',this.videos[index]);
                    await this.createVideo(this.videos[index])
                    
                   }

                    
                    
                    this.$refs.myform.reset();
                    this.successAlert = true;
                  }else {
                   this.errorAlert = true;
                  }
              
              
          },

          async createVideo(video){
            

            try {

              console.log("line 227 ");
              const videoTitle=video.name.substring(0,video.name.lastIndexOf("."));
              let videoPath = null;
              const resp = await http.postMedia("/admin/file/create",video);
              console.log('status',resp.status);
              if (resp.status === 200) {
                  videoPath = await resp.text();
                  console.log('videot path ',videoPath,'video naem : ',videoTitle);
                  console.log("course ",this.createdCourse);

                  await axios.post(constant.localDomain+'/admin/createCourseVideo',{
                    title:videoTitle,
                    path:videoPath,
                    course:{id: this.createdCourse.id},
                  }).then((Response)=>{
                    console.log("rsponse ",Response);
                  })
              }
              else {
                  this.errorAlert = true;
                  console.log("error while vido add ",resp);
              }
               
              
            } catch (error) {
              console.log('error');
            }
            
              
              



          },
          async showfiles(){
            console.log("files : ",this.videos);
            for (let index = 0; index < this.videos.length; index++) {
                    console.log('current video :',this.videos[index]);
                    
                   }
          },
          async removeVideo(){

          },
          async onChangeImage(imageFile){
            this.imagePreviewPath=null;
            this.imagePreviewPath = URL.createObjectURL(imageFile);

          }


      },
      
  }
  </script>