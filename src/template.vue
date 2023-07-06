<script lang="ts">
import { defineComponent } from "vue";
import type { Ref } from "vue";

export default defineComponent({
  data() {
    return {
      myData: "Hello world!",
    };
  },
  methods: {
    fetchData(): void {
      // Perform some async operation to fetch data
      this.myData = "Data fetched successfully!";
    },
  },
  mounted() {
    this.fetchData();
  },
});
</script>

<template>
  <div>{{ myData }}</div>
</template>
