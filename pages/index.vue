<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8 md6>
      <QrReader></QrReader>
    </v-flex>
  </v-layout>
</template>

<script>
import QrReader from "~/components/QrReader.vue";

export default {
  components: {
    QrReader
  }
};
</script>
