<template>
  <v-container fluid>
    <div v-if="loading">laoding content</div>

    <div v-else>
      <v-row>
        <v-col>
          <v-card class="mx-auto" width="auto">
            <v-card-title>{{ knowledgeList[currentIndex].title }}</v-card-title>
            <v-card-subtitle >
              posted on : {{ knowledgeList[currentIndex].date }}
            </v-card-subtitle>
            <v-card-subtitle >{{ knowledgeList[currentIndex].tag }}</v-card-subtitle>

            <v-card-text v-for="(item, index) in contentArray" :key="index">
              <div v-if="item.type === 'text'" v-html="item.content"></div>
              <v-img
                v-else-if="
                  item.type === 'image' && docxContent.images.length >= item.id
                "
                :src="docxContent.images[item.id - 1]"
                alt="Image"
                :draggable="true"
              ></v-img>
            </v-card-text>
            <v-card-actions>  </v-card-actions>
          </v-card>

          <v-btn @click="previous()"> previous</v-btn>
          <v-btn @click="next()"> next</v-btn>
          
         
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>

<script>
import utils from "../utils/utils";
import http from "../utils/http";
import DocxContentDisplay from "./DocxContentDisplay.vue";
import router from "@/router";
import axios from "axios";
import constant from "@/utils/constant";

export default {
  name: "knowledge_detail",
  components: {
    DocxContentDisplay,
  },

  data() {
    return {
      localDomain: utils.constant.localDomain,
      knowledgeList:[],
      currentIndex:this.$route.params.index,
      contentArray: [],
      docxContent: null,
      loading: true,
    };
  },

  async created() {
    // LoginUser from Vuex

    //await this.fetchKnowledge();
    await this.fetchKnowledges();
  },

  methods: {
    // async fetchKnowledge() {
    //   await http
    //     .get(this.localDomain + "/knowledge/" + this.knowledge_id)
    //     .then((Response) => {
    //       this.knowledge = Response.data;
    //       this.fetchDocx();
    //     });
    // },
    async fetchKnowledges() {
      axios.get(utils.constant.get_knowledge_url).then((Response) => {
        this.knowledgeList = Response.data;
        this.fetchDocx();
      });
    },
 
    async fetchDocx() {
      //const currentKnowledege= this.knowledgeList[this.currentIndex];
      console.log('list',this.knowledgeList);
      
      console.log('file name :',this.knowledgeList[this.currentIndex].contentPath);
        await axios.get(constant.localDomain+"/admin/readDocx",{
          params:{
            fileName: this.knowledgeList[this.currentIndex].contentPath,
          }
        })
        .then((Response) => {
          this.docxContent = Response.data;
          this.splitContent();
        });
    },
    splitContent() {
      const lines = this.docxContent.textContent.split("\n");
      var imageindex = 0;
      this.contentArray = lines.map((line) => {
        if (line.startsWith("IMAGE:")) {
          imageindex++;
          console.log("var", imageindex);
          return { type: "image", id: imageindex };
        } else {
          return { type: "text", content: line };
        }
      });
      
      this.loading = false;
      window.scrollTo({
        top: 0,
        behavior: 'auto' // Optional: Adds smooth scrolling animation
      });
    },

  
    next() {
  if (this.currentIndex < this.knowledgeList.length - 1) {
    this.currentIndex++;
    //this.loading = true;
    //this.fetchDocx();
    router.push({ path: "/knowledge_detail/" + this.currentIndex});
  }
  else{
    console.log("last endex")
  }
},
previous() {
  if (this.currentIndex > 0) {
    this.currentIndex--;
    //this.loading = true;
    //this.fetchDocx();
    router.push({ path: "/knowledge_detail/" +this.currentIndex});
  }
  else{
    console.log("smallest endex")
  }
},
  },
};
</script>

<style>
.centered-div {
  /* Set the custom size for the div */
  width: auto;

  /* Center the div horizontally and vertically */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  /* Add other styling if needed */
}
</style>
