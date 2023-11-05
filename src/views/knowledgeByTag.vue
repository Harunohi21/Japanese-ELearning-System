<template>
    <div>
      <v-select 
        v-model="selectedTopic"
        :items="topics"
        label="TOPICS"
        solo
        @change="showbyCategory"
      ></v-select>
   
    <div v-if="knowledgeList.length<1"> no post avaliable </div>
    <div v-else>
      <div class="text-center">
      
    </div>
      <v-col>
  
        <v-row   v-for="knowledge in knowledgeList" :key="knowledge.id">
          <v-col>
            <v-card  @click="goToKnowledgeDetail(knowledge)">
              <v-card-title>
                {{knowledge.title}}
              </v-card-title>
              <v-card-subtitle>
                {{ knowledge.tag }} | {{ knowledge.date }}
              </v-card-subtitle> 
              <v-card-text >  {{knowledge.intro}}</v-card-text>
  
                
              
            </v-card>
          </v-col>
            
            
        
        </v-row>
      </v-col>
    </div>
  </div>
  </template>
  
  <script>
  import constant from "@/utils/constant";
  import utils from "../utils/utils";
  import axios from "axios";
import router from "@/router";
  
  export default {
    name: "knowledge",
  
    components: {},
  
    data() {
      return {
        selectedTopic:"",
        topics:[
            "all",
          "anime",
          "food",
          "history",
          "culture",
          
  
        ],
       
        localDomain: utils.constant.localDomain,
        knowledgeList: [],
        tag:this.$route.params.tag,
      };
    },
  
    async created() {
      await this.fetchKnowledges();
    },
    
    methods: {
      async fetchKnowledges() {
        axios.get(this.localDomain+'/knowledge/tag',{
          params:{
            tag:this.tag
          }
        }).then((Response) => {
          this.knowledgeList = Response.data;
        });
        console.log('list',this.knowledgeList)
      },
   
      goToKnowledgeDetail(knowledge) { 
        this.$router.push({
          path: "/knowledge_detail_tag/"+this.tag +"/"+this.knowledgeList.indexOf(knowledge),
        });
      },
  
      async showbyCategory(){
      console.log(this.selectedTopic);
      if(this.selectedTopic=="all"){
        console.log("all")
      this.$router.push("/knowledge")
      }else{
        this.$router.push("/knowledgeByTag/"+this.selectedTopic);
      }
      
    }
      
    },
  };
  </script>
  
  <style scoped>
  .container {
    margin-left: 10vw !important;
    margin-right: 10vw !important;
  }
  </style>
  