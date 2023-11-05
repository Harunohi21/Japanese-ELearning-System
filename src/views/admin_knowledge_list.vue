<template>
  <div>
    <v-row>
      <!-- Sidebar -->
      <v-col cols="3">

      <admin_sidebar></admin_sidebar>
    </v-col>
    <v-col cols="8">

  <v-data-table
    :headers="headers"
    :items="knowledgeList"
    :items-per-page="5"
    sort-by="title"
    class="elevation-3"
    
    
  >
  
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>Knowledge Posts</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
      </v-toolbar>
    </template>
   
    <template v-slot:item.actions="{ item }">
      <v-icon
        small
        class="mr-2"
        @click="onClickUpdateItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="toDeleteItem=item;
                dialogDelete=true;
        "
      >
        mdi-delete
      </v-icon>
    </template>

  </v-data-table>
  <!-- Delete Dialog -->
  <v-dialog v-model="dialogDelete" width="400"
  >
      <v-card>
        <!-- Dialog Heading -->
        <v-toolbar color="orange" dark>
          <div>Delete this Knowledge Post ?</div>
          <v-spacer></v-spacer>
          <v-btn icon @click="toDeleteItem={};
                              dialogDelete = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>

        <!-- Delete Knowledge Info -->
        <v-card-text class="pa-4">
          <v-row dense>
            <v-col cols="3" class="font-weight-bold">Ttile</v-col>
            <v-col cols="7">{{ toDeleteItem.title }}</v-col>
            <v-col cols="3" class="font-weight-bold">Topic</v-col>
            <v-col cols="7">{{ toDeleteItem.tag }}</v-col>
            <v-col cols="3" class="font-weight-bold">Introduction</v-col>
            <v-col cols="7">{{ toDeleteItem.intro }}</v-col>
          </v-row>
        </v-card-text>

        <!-- Delete knowledge Btn -->
        <v-card-actions class="justify-end">
          <v-btn  class="ma-2"  @click="toDeleteItem={}; dialogDelete = false">Cancel</v-btn>
          
          <v-btn color="red"  @click="deleteKnowledge()">Confirm</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Update Dialog -->
    <v-dialog v-model="dialogUpdate" width="500" persistent >
      <v-card>
        <!-- Dialog Heading -->
        <v-toolbar color="primary" dark>
          <div>Update This Knowledge?</div>
          <v-spacer></v-spacer>
          <v-btn icon @click="dialogUpdate = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>

        <!-- Update Form -->
        <v-card-text class="pa-4">
          <v-form ref="knowledgeForm" v-model="knowledgeForm">
            <!-- Update Movie Title -->
            <v-text-field
              v-model="toUpdateItem.title"
              :counter="50"
              :rules="[
                (v) => !!v || 'Required',
                (v) =>
                  (v && v.length <= 50) ||
                  'Title must be less than 50 characters',
              ]"
              label="Title"
              required
            ></v-text-field>

            <!-- Update knowledge topic -->
            <v-select 
      v-model="toUpdateItem.topic"
      multiple
       dense
      outlined
      required
      :items="['anime', 'food', 'culture', 'language', 'history']"
      :rules="[(v) => v.length>0 || 'Please select at least one item']"
      label="select topic"
      
    ></v-select>

            <!-- Update knowledge introduction -->
            <v-textarea
              v-model="toUpdateItem.introduction"

            
              :counter="500"
              :rules="[
                (v) => !!v || 'Required',
                (v) =>
                  (v && v.length <= 500) ||
                  'Title must be less than 500 characters',
              ]"
              label="Introduction"
              required
            ></v-textarea>


            <!-- Update knowledge content file -->
            <v-file-input
              v-model="toUpdateItem.file"
              label="Content File"
              show-size
              prepend-icon="mdi-file-word"
              placeholder="Choose content file"
              accept=".docx"
              :rules="[
                (v) =>
                  !v ||
                  v.size < 20000000 ||
                  'file size should be less than 20 MB!',
              ]"
              
            ></v-file-input>

            <!-- Update Btn -->
            <v-btn
              :disabled="!knowledgeForm"
              color="success"
              class="mr-4"
              @click="updateKnowledge()"
            >
              <span v-if="!loading">Update</span>
              <v-progress-circular
                v-else
                indeterminate
                color="primary"
              ></v-progress-circular>
            </v-btn>

            
          </v-form>
        </v-card-text>
      </v-card>
    </v-dialog>
    <!-- Success Alert -->
    <v-expand-transition>
    <v-alert class="mt-3" v-show="successAlert" dense type="success"
              dismissible
              >
              Update knowledge success!
            </v-alert>
          </v-expand-transition>
            <!-- Error Alert -->
            <v-alert class="mt-3" v-show="errorAlert" dense type="error" 
            dismissible>
              Update knowledge Failed!
            </v-alert>
    
    
          </v-col>
 </v-row>
  </div>

</template>
     
     <script>
     import admin_sidebar from '../components/admin_sidebar.vue'
     import constant from '@/utils/constant';
import http from '@/utils/http';
     import axios from 'axios';
     
     export default {
        name:'admin_knowledge_list',
        components:{admin_sidebar},
       data() {
         return {
          loginUser:{},
            
      headers: [
        {
          text: 'title',
          align: 'start',
          sortable: false,
          value: 'title',
        },
        
        { text: 'topic', value: 'tag',sortable: false  },
        { text: 'file ', value: 'contentPath',sortable: false  },
        { text: 'introduction', value: 'intro' ,sortable: false },
        
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      knowledgeList:[],
      knowledgeForm: false,
      toDeleteItem:{},
      dialogDelete: false,

      dialogUpdate: false,
      toUpdateItem: {
        id:null,
        title: '',
        introduction: '',
        topic:[],
        file:null,
      
      },
      contentPath:null,
      loading:false,
      errorAlert:false,
      successAlert:false,
      
      
         };
       },

    created () {
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
      this.fetchKnowledges()
  
    },

    methods: {
      async fetchKnowledges() {
      axios.get(constant.get_knowledge_url).then((Response) => {
        this.knowledgeList = Response.data;
      });
      console.log('list',this.knowledgeList);
    },
    async deleteKnowledge(){
      
        await axios.delete(constant.localDomain+'/admin/knowledge/delete/'+this.toDeleteItem.id)
        .then((Response)=>{
          if(Response.status==200){
            this.dialogDelete=false;
            
          }else{
            this.errorAlert=true;
          }
        })
        await this.fetchKnowledges();
    },
    async onClickUpdateItem(item){
     
      this.dialogUpdate=true;
      //this.toUpdateItem = Object.assign({}, item);
      this.toUpdateItem.id=item.id;
      this.toUpdateItem.title=item.title;
      this.toUpdateItem.introduction= item.intro;
      this.toUpdateItem.topic=item.tag.split(',');    
      this.contentPath=item.contentPath;    
    },
    async updateKnowledge(){
      this.loading=true;
     

      if (this.toUpdateItem.file!=null){
        
          console.log('new file ',this.contentPath);  
      const resp=await http.putMedia('/admin/file/update',this.toUpdateItem.file,this.contentPath);
      
            if (resp.status === 200) {
                this.contentPath = await resp.text();
                
            }
            else {
                console.log("error : file update failed");
            }
          
        
        
      }
        
      const respKnowledge=   await axios.put(constant.localDomain + "/admin/knowledge/update/"+
                  this.toUpdateItem.id, 
                {
                   title: this.toUpdateItem.title,
                  tag: this.toUpdateItem.topic.join(","),
                  contentPath:this.contentPath,
                  intro: this.toUpdateItem.introduction,
                })
            if(respKnowledge && respKnowledge.status==200){
              this.contentPath=null;
              this.dialogUpdate=false;
              this.toUpdateItem.file=null;
              
              await this.fetchKnowledges();
              
              setTimeout(() => {
                this.successAlert=true;
              }, 500);
              setTimeout(() => {
                this.successAlert=false;
              }, 10000);
            }else{
              this.errorAlert=true;
            }
       this.loading=false;    
    
     },
    }
     
     };
     </script>
     <style scoped>
     .colorful-table{
       background-color: #f69e07;
     }
     
     /* Apply a predefined color class to the table rows on hover */
     .colorful-table tbody tr:hover {
       background-color:aqua;
     }
     </style>
     