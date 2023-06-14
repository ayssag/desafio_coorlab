<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>
    <BestTransportAnalysis @userInput="bestAnalysis" :cities="cities"></BestTransportAnalysis>
    <hr>
    <div v-if="minCostCompany.id && fastCostCompany.id">
      <b-card title="Menor Custo" style="background-color: #CCCCFF; margin: 25px 25px 25px 25px;">
        {{ minCostCompany.name }}<br>
        Tempo de Entrega: {{ minCostCompany.lead_time }}
        <hr>
        Custo Total: R$ {{ minCostCompany.total_cost.toFixed(2) }}
      </b-card>
      <b-card title="Menor Tempo de Entrega" style="background-color: #9FE2BF; margin: 25px 25px 25px 25px;">
        {{ fastCostCompany.name }}<br>
        Tempo de Entrega: {{ fastCostCompany.lead_time }}
        <hr>
        Custo Total: R${{ fastCostCompany.total_cost.toFixed(2) }}
      </b-card>
    </div>
  </div>
</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'
import axios from 'axios';
import BestTransportAnalysis from './BestTransportAnalysis.vue';

export default {
  components: {
    BNavbar,
    BNavbarBrand,
    BestTransportAnalysis
  },
  data() {
    const appName = ''

    return {
      appName,
      transports: [],
      minCostCompany: {},
      fastCostCompany: {},
      cities: []
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    this.appName = 'Melhor Frete';
    axios
      .get("http://localhost:3000/transport")
      .then((res) => {
        this.transports = res.data;
        this.cities = this.transports.map(transport => transport.city);
        this.cities = [...new Set(this.cities)];
      });

  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      console.log(this.appName)
    },
    getCityCompanies(companies, cityName) {
      let companiesByCity = [];
      companies.forEach(company => {
        if (company['city'] == cityName) {
          companiesByCity.push(company);
        }
      })
      return companiesByCity;
    },
    formatCost(cost) {
      let lengthCost = cost.length;
      cost = cost.slice(3, lengthCost);
      cost = Number(cost);

      return cost;
    },
    calcMinCost(companies, weight) {
      let minCost = Infinity;
      let companyCost = undefined;
      let minCostCompany = companies[0];
      let paramName = weight <= 100.0 ? 'cost_transport_light' : 'cost_transport_heavy';
      minCost = this.formatCost(companies[0][paramName]);
      companies.forEach(company => {
        companyCost = this.formatCost(company[paramName]);
        if (companyCost < minCost) {
          minCost = companyCost;
          minCostCompany = company;
        }
      });

      return minCostCompany;
    },
    formatLeadTime(leadTime) {
      let lengthLeadTime = leadTime.length;

      leadTime = leadTime.slice(0, lengthLeadTime - 1);
      leadTime = Number(leadTime);

      return leadTime;
    },
    calcFastCost(companies) {
      let companyLeadTime = Infinity;
      let fastCost = this.formatLeadTime(companies[0].lead_time);
      let fastCostCompany = companies[0];
      companies.forEach(company => {
        companyLeadTime = this.formatLeadTime(company.lead_time);
        if (companyLeadTime < fastCost) {
          fastCost = companyLeadTime;
          fastCostCompany = company;
        }
      });
      return fastCostCompany;
    },
    bestAnalysis(userInput) {
      let destiny = userInput.destiny;
      let weight = userInput.weight;
      let companiesByCity = this.getCityCompanies(this.transports, destiny);

      this.minCostCompany = this.calcMinCost(companiesByCity, weight);
      this.fastCostCompany = this.calcFastCost(companiesByCity);

      this.minCostCompany['total_cost'] = weight <= 100.0 ? (this.formatCost(this.minCostCompany.cost_transport_light)) 
        : (this.formatCost(this.minCostCompany.cost_transport_heavy));
      this.minCostCompany['total_cost'] *= weight;

      this.fastCostCompany['total_cost'] = weight <= 100.0 ? (this.formatCost(this.fastCostCompany.cost_transport_light)) 
        : (this.formatCost(this.fastCostCompany.cost_transport_heavy));
      this.fastCostCompany['total_cost'] *= weight;
    },
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
</style>
