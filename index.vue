<template>
  <time-line :options="options" :sourceData="data"></time-line>
</template>

<script>
import TimeLine from "./TimeLine";

const data = [
  {
    start: "2020-06-01",
    end: "2020-06-05",
    text: "渠道1",
  },
  {
    start: "2020-06-06",
    end: "2020-06-07",
    text: "渠道2",
  },
  {
    start: "2020-07-01",
    end: "2020-07-20",
    text: "渠道4",
  },
  {
    start: "2020-06-15",
    end: "2020-06-25",
    text: "渠道3",
  },
  {
    start: "2020-07-23",
    end: "2020-08-01",
    text: "渠道5",
  },
];
export default {
  components: {
    "time-line": TimeLine,
  },
  data() {
    return {
      data,
      options: {
        width: 500,
        height: 200,
      },
    };
  },
};
</script>

<style></style>
