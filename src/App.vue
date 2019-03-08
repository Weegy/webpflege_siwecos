<template>
  <div id="app">
    <input type="text" class="form-control" id="domain" v-model="domain" />
    
    <button class="btn btn-success" @click="enterDomain">Abschicken</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import axios from 'axios';
import { SiwecosRequest } from './interface/siwecosrequest';

@Component({
  components: {
  },
  data() {
    return {
      domain: '',
      requestId: 0,
      result: {}
    };
  },
})
export default class App extends Vue {
  public enterDomain() {
    const siweocsData: SiwecosRequest = {domain: this.$data.domain};
    axios.post('https://ca.siwecos.de/api/v1/getFreeScanStart', siweocsData).then((result) => {
      this.$data.requestId = result.data.id;
      this.pollRequest();
    });
  }

  public async pollRequest() {
    const self = this;
    await setTimeout(() => {
        axios.get(`https://ca.siwecos.de/api/v1/scan/status/free/${self.$data.requestId}`).then((result) => {
        if (result.data.progress === 100) {
          self.processResults();
        } else {
          self.pollRequest();
        }
      });
    }, 2000);
  }

  public processResults() {
    axios.get(`https://bla.siwecos.de/api/v1/freescan/result/${this.$data.requestId}/de`).then((result) => {
      this.$data.result = result.data;
      console.log(this.$data.result); 
    })
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
