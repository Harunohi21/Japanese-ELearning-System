<!-- DocxContentDisplay.vue -->
<template>
    <div>
      <!-- Display the text content -->
      <div v-html="docxContent.textContent"></div>
  
      <!-- Display images using Base64 src -->
      <div v-for="image in docxContent.images" :key="image">
        <img :src="image" alt="Embedded Image" />
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      docxContent: {
        type: Object,
        required: true,
      },
    },
  };
  </script>
  
  <style>
  /* Add any necessary styles here */
  </style>
  