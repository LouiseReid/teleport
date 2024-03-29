<template>
  <div id="cost-container">
    <div v-if="cities.length === 0" id="loading">
      <v-progress-circular indeterminate color="#829fce"/>
    </div>
    <v-container grid-list-md v-else id="cost-container--inner">
        <slot name="nav"/>
      <v-layout align-center justify-center row>
        <input
          type="text"
          v-model="search"
          @keyup="searchForCity"
          placeholder="search for city..."
          id="city-search"
        >
      </v-layout>
      <v-layout row wrap v-if="!search">
        <v-flex xs6 md3 v-for="(city, index) in cities" :key="index" >
          <v-card class="ma-2 pa-3" color="#e7e9ea">
            <v-card-text primary-title class="pa-2">
              <h3 class="text-capitalize text-sm-center">{{city.name}}</h3>
            </v-card-text>
            <v-card-text
              v-for="(category, index) in city.categories[0].data"
              :key="index"
              class="pa-0 mb-0"
            >
              <div v-if="category.label !== 'Inflation [Teleport score]'">
                <p class="mb-1 text-sm-center text-xs-center" secondary-title>
                  <strong>{{category.label}}:</strong>
                  ${{category['currency_dollar_value']}}
                </p>
              </div>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
      <v-layout v-else>
        <v-card class="ma-2 pa-4" color="#e7e9ea">
          <v-card-text primary-title class="pa-2">
            <h3 class="text-capitalize text-sm-center">{{searchedCity.name}}</h3>
          </v-card-text>
           <v-card-text
              v-for="(category, index) in searchedCity.categories[0].data"
              :key="index"
              class="pa-0 mb-0"
            >
              <div v-if="category.label !== 'Inflation [Teleport score]'">
                <p class="mb-1 text-sm-center text-xs-center" secondary-title>
                  <strong>{{category.label}}:</strong>
                  ${{category['currency_dollar_value']}}
                </p>
              </div>
            </v-card-text>
        </v-card>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import EventService from "../services/EventService.js";

export default {
  data() {
    return {
      cities: [],
      search: "",
      searchedCity: null
    };
  },
  methods: {
    searchForCity() {
      let foundCity = this.cities.find(city => {
        return city.name.toLowerCase().indexOf(this.search.toLowerCase()) > -1;
      });
      this.searchedCity = foundCity;
    }
  },
  mounted() {
    EventService.getCosts()
      .then(res =>
        res.map(city => ({
          name: city.config.url
            .split("/")
            .slice(-3)[0]
            .replace("slug:", "")
            .replace(/-/g, " "),
          categories: city.data.categories.filter(
            category => category.label === "Cost of Living"
          )
        }))
      )
      .then(cities => (this.cities = cities));
  }
};
</script>

<style lang="scss" scoped>
#cost-container {
  background-image: url("../assets/cost_background.jpg");
  background-size: cover;
  position: sticky;
  &--inner {
    height: 100vh;
    width: 100vw;
    overflow-y: scroll;
  }
}

#loading {
  background: white;
  width: 100vw;
}

strong {
  margin-right: 5px;
}

#city-search {
  border: 1px solid #797878;
  padding: 5px;
  background-color: #e7e9eae5;
}
</style>