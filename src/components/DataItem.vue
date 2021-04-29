<template>
  <div>
    {{ JSON.stringify(dataItem) }}
  </div>
</template>

<script>
// import utils from "@/helpers/utils";

export default {
  name: "DataItem",
  props: {
    dataItem: Object,
  },
};
</script>
