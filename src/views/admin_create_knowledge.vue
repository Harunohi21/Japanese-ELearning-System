<template>

  <div>
    <v-row>
      <!-- Sidebar -->
      <v-col cols="3">

      <admin_sidebar></admin_sidebar>
    </v-col>
    <v-col cols="8">
    
  <v-sheet class="mx-auto"  >
    
    <v-form   ref="myform" fast-fail @submit.prevent >
      <v-text-field
        v-model="title"
        outlined
        dense
        label="title"
        :rules="titleRules"
      ></v-text-field>

      <v-textarea
      counter
      outlined
      label="Introduction"
      :rules="introRules"
      v-model="intro"
    ></v-textarea>

    
    <v-select 
      v-model="topics"
      multiple
       dense
      outlined
      :items="['anime', 'food', 'culture', 'language', 'history']"
      :rules="topicRules"
      label="select topic"
      required
      
    ></v-select>

   
  <v-file-input 
    show-size
    outlined
    dense
    prepend-icon=''
    append-icon="mdi-file-word"
      v-model="contentFile"
    accept=".docx"
    label="upload content file"
    @change="generatetitle"
    
  ></v-file-input>
  
      <v-btn  block class="mt-2" @click="submit" :disabled="!iscomplete">Submit</v-btn>
      <v-alert type="success"
  v-model="successAlert"
  dismissible
  >
      knowledge created !
    </v-alert>
  <v-alert class="mt-3" v-show="errorAlert" dense type="error">
            Create knowledge Failed !
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

    name:'checkout',
    components:{admin_sidebar},
    data: () => ({
        contentFile: null,
        title: "",
        titleRules: [
            value => {
                if (value?.length > 3)
                    return true;
                return "Title  must be at least 3 characters.";
            },
        ],
        topics: [],

        topicRules:[(v) => v.length>0 || 'Please select at least one item'],
        intro: "",
        introRules: [
            value => {
                if (value?.length > 10) {
                    if (value?.length > 500)
                        return "not more than 500";
                    return true;
                }
                return "Introduction must have at least 10";
            }
        ],
        errorAlert: false,
        successAlert: false,
        
    }),
    computed: {
        iscomplete() {
            return this.title && this.intro && this.contentFile != null && this.topics != null;
        },
    },
    methods: {

        async generatetitle() {
            console.log("file : ", this.contentFile);
            //this.shortContent.
            console.log("video title :", this.contentFile.name.substring(0, this.contentFile.name.lastIndexOf(".")));
        },
        async submit() {
            let contentPath = null;
            const resp = await http.postMedia("/admin/file/create", this.contentFile);
            console.log('resp staus',resp.status,'resp text ');
            if (resp.status === 200) {
              
                contentPath = await resp.text();
            }
            else {
                this.errorAlert = true;
            }
            try {
                await axios.post(constant.localDomain + "/admin/knowledge/create ", {
                    title: this.title,
                    tag: this.topics.join(","),
                    contentPath: contentPath,
                    intro: this.intro,
                }).then((Response) => {
                    console.log("created : ", Response.data);
                    this.$refs.myform.reset();
                    this.successAlert = true;
                });
            }
            catch (error) {
              this.errorAlert=true;
                console.log("error while creating content ", error);
            }
        }
    },
    
}
</script>