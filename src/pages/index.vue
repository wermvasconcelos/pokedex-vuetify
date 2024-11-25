<template>
  <!-- <HelloWorld /> -->
</template>

<script lang="ts" setup>
  //
  import axios from 'axios';

  axios({
    method: 'get',
    url: 'https://pokeapi.co/api/v2/pokemon/ditto',
    responseType: 'json',
  }).then((response) => {
    console.log(response.data)
  })
</script>
