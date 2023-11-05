<template>
  
    <v-card width="250">
      
        <v-navigation-drawer
          permanent
         
          
        >
        <v-list-item >
              <v-list-item-content>
                <v-list-item-title class="text-h6">
                  Admin Dashboard
                </v-list-item-title>
                <v-list-item-subtitle>Manage </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
           
          <v-divider></v-divider>
  
          <v-list dense nav>
        <v-list-item
          v-for="item in menuItemList"
          :key="item.title"
          link
          @click="goToRoute(item.path)"
        >
          <!-- Icon -->
          <v-list-item-icon>
            <svg-icon type="mdi" :path="item.icon"></svg-icon>
          </v-list-item-icon>

          <!-- Title -->
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
        </v-navigation-drawer>     
      </v-card>
  </template>

<script>
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiViewList } from '@mdi/js';
import { mdiFileDocumentPlus } from '@mdi/js';
import { mdiFilePlus } from '@mdi/js';
import { mdiAccountMultiple,mdiAccountSupervisor } from '@mdi/js';
export default {
  name: 'admin_sidebar',
  components:{SvgIcon},

  data: () => ({
    menuItemList: [
    

    {
        title: "Course List",
        icon: mdiViewList,
        path: "/admin",
      },
      {
        title: "Create Course",
        icon: mdiFilePlus,
        path: "/admin/course/create",
      },
      {
        title: "Knowledge Posts",
        icon: mdiViewList,
        path: "/admin/knowledge_list",
      },
      {
        title: "Create Knowledge",
        icon: mdiFileDocumentPlus,
        path: "/admin/knowledge/create",
      },
      {
        title: "Admin List",
        icon: mdiAccountSupervisor,
        path: "/admin/admin_list",
      },
      {
        title:"User List",
        icon: mdiAccountMultiple,
        path: "/admin/user_list",
      }
     
      

      
    ],
  }),

  //

  methods:{
    goToRoute(path) {
      // If Current Path is same with Clicked Path, No Go to Route
      if (this.$route.path != path) {
        this.$router.push({ path: path });
      }
    },
  }
}
</script>