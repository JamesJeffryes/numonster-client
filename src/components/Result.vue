<template>
  <div id='result' class='container main'>
    <h3>
      Results for {{$route.params.job_id}}
    </h3>
    <table class="table table-striped">
      <thead>
      <tr>
        <th>Model</th>
        <th>Chain 1</th>
        <th>Chain 2</th>
        <th colspan="2">View Results</th>
        <th>PDB</th>
        <th>Results</th>
        <th>Logs</th>
      </tr>
      </thead>
      <tbody v-if="result">
        <template v-for="(chains, model, index) in result.models">
          <template v-for="(links, chain, index2) in chains">
            <tr>
              <td>{{model}}</td>
              <td>{{chain.slice(0,1)}}</td>
              <td>{{chain.slice(1,2)}}</td>
              <td><button class="button btn-default" v-on:click='open2D()'>2D Display</button></td>
              <td><button v-if="links.PDB" class="button btn-default" v-on:click='open3D(links.PDB.PDB, chain.slice(0,1), chain.slice(1,2))'>3D Display</button></td>
              <td><a v-if="links.PDB" v-bind:href="links.PDB.PDB">PDB File</a></td>
              <td>
                <a target="_blank" v-bind:href="links.Results.XML">XML File</a>
                <br>
                <a target="_blank" v-bind:href="links.Results.TXT">TXT File</a>
              </td>
              <td>
                <a target="_blank" v-if="links.Logs" v-bind:href="links.Logs.MSMS">MSMS</a>
                <br>
                <a target="_blank" v-if="links.Logs" v-bind:href="links.Logs.HBPlus">HBPlus</a>
              </td>
            </tr>
          </template>
        </template>
      </tbody>
    </table>
    <div class="alert alert-danger" v-if="!result">
      Fetching results
    </div>
  </div>
</template>

<script>
export default {
  name: 'Result',
  data: () => ({
    result: "",
  }),
  mounted: function () {
    this.$http.get(this.$server_url + '/results/' + this.$route.params.job_id).then(function (response) {
      console.log(response);
      this.result = response.body;
    }, function (response) {
      console.error(response);
      alert('File job fetch failed. Check your browser console for details');
    });
  },
  methods: {
    open3D: function (pdb_url, chain1, chain2) {
      window.open(`http://3dmol.csb.pitt.edu/viewer.html?url=${pdb_url}
      &surface=opacity:0.6&select=chain:${chain1}&style=cartoon:color~green
      &select=chain:${chain2}&style=cartoon:color~red`)

    },
    open2D: function () {
      alert('Not yet implemented')
    }
  }
}
</script>

<style scoped>
</style>
