<template>
  <!-- first chat for dsiplaying number of people with the same bloodgroup -->
  <highcharts :options="chartOptions"></highcharts>

  <!-- second chart for dsiplaying number of people that fall in the same age range -->
  <highcharts :options="agerange"></highcharts>
</template>

<script>
import axios from "axios";

import { Chart } from "highcharts-vue";
export default {
  name: "App",

  components: {
    highcharts: Chart,
  },

  data() {
    return {
      chartOptions: {
        chart: {
          type: "bar",
        },

        title: {
          text: "Blood Group",
        },

        /**blood group that are unique */
        xAxis: {
          title: {
            text: "Blood Group",
          },
          categories: [],
        },

        yAxis: {
          title: {
            text: "Number of People",
          },
        },

        series: [
          {
            name: "Number of People",
            data: [], //age
          },
        ],
      },

      // data for secondchart
      agerange: {
        chart: {
          type: "bar",
        },

        title: {
          text: "Age Range",
        },

        /**Age group values */
        xAxis: {
          title: {
            text: "Age Range",
          },

          // when the person age falls between the range of 1-17-->Child
          // when the person age falls between the range of 18-29-->Young Adult
          // when the person age falls between the range of 29 and above-->Adult

          categories: ["Child", "Young Adult", "Adult"],
        },

        yAxis: {
          title: {
            text: "Number of People",
          },
        },

        series: [
          {
            name: "Number of People",
            data: [], //number of people who fell in the same age group
          },
        ],
      },
    };
  },
  mounted() {
    this.getdata();

    // return unique characters in
    // console.log(new Set(this.chartOptions.xAxis.categories));
  },

  methods: {
    /**Api request to get data */
    getdata() {
      axios
        .get(
          "https://frontend-vuejs-assignmen-eeb46-default-rtdb.firebaseio.com/sampledata.json"
        )
        .then((response) => {
          // Storing fetched data in localStorage by storing it as an object

          localStorage.setItem("Bloodgroup", JSON.stringify(response.data));

          //retrieving it from local storage
          let bloodgroupdata = localStorage.getItem("Bloodgroup");

          //converting and returning it back as a string
          let convertedbloodgroupdata = JSON.parse(bloodgroupdata);
          this.formatdata(convertedbloodgroupdata);
        });
    },

    /**format the data */
    formatdata(data) {
      /**array containing preformatted values of data */
      let bloodgroup = [];

      /**array that will contain formatted values of data for only field bloodgroup */
      let newbloodgroup = [];

      /**array that will contain formatted values of data for only field age */
      let age = [];
      for (let key in data) {
        bloodgroup.push({ ...data[key] });
      }

      /**formatting it and selecting only the bloodgroup and age  */
      for (let k in bloodgroup) {
        newbloodgroup.push(bloodgroup[k].bloodtype);

        /**This will contain values for the age of the people in the data returned */
        age.push(bloodgroup[k].age);
      }

      /**setting our bloodgroup to return unique values */
      let c = 0;
      let start = false;

      for (let i = 0; i < newbloodgroup.length; i++) {
        for (let j = 0; j < this.chartOptions.xAxis.categories.length; j++) {
          if (newbloodgroup[i] == this.chartOptions.xAxis.categories[j]) {
            start = true;
          }
        }
        c++;

        if (c == 1 && start == false) {
          this.chartOptions.xAxis.categories.push(newbloodgroup[i]);
        }
        start = false;
        c = 0;
      }

      //count number of bloodgroup that repeated = to the number of people that has that same blood group
      let number_of_ppl_for_bloodgroupcount = {};

      for (let counter of newbloodgroup) {
        if (number_of_ppl_for_bloodgroupcount[counter]) {
          number_of_ppl_for_bloodgroupcount[counter] += 1;
        } else {
          number_of_ppl_for_bloodgroupcount[counter] = 1;
        }
      }

      //move set containing the number of bloodgroup that repeated into the data array for bar chart
      this.chartOptions.series[0].data = Object.values(number_of_ppl_for_bloodgroupcount);
      console.log(this.chartOptions.series[0].data);

      /**Filter to return totalcount of people that are between the ages of 1-17 = Child*/

      let one_to_seventeen = age.filter((obj) => {
        if (obj <= 1 || obj <= 17) {
          return true;
        }

        return false;
      }).length;

      /**Filter to return totalcount of people that are between the ages of 18-29 = Young Adult*/
      let eighteen_to_twentynine = age.filter((obj) => {
        if (obj >= 18 && obj <= 29) {
          return true;
        }

        return false;
      }).length;

      /**Filter to return totalcount of people that are between the ages of 30 upwards = Adult*/
      let thrithyupwards = age.filter((obj) => {
        if (obj >= 30) {
          return true;
        }

        return false;
      }).length;

      /**push the returned number of people for each age group
       * children
       * young adults
       * adults
       */
      this.agerange.series[0].data.push(
        one_to_seventeen,
        eighteen_to_twentynine,
        thrithyupwards
      );
    },
  },
};
</script>
