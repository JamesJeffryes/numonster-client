<template>
  <div id='result' class='container main'>
    <div v-if="display_graph" class="card">
    <div class="card-header">
      <span>{{display_graph}}</span>
      <button-close v-on:click="close2D()" class="btn"></button-close>
    </div>
    <div class="row card-body">
      <div  class="col-10">
        <div id="graph-container"></div>
      </div>
      <div class="col-2">
        <br>
        <div id="color-table">
          <table>
            <thead>Edge Types</thead>
              <tr v-for="(color, edge) in edge_colors" :key="edge">
                <td><span v-bind:style="{color: color, 'background-color': 'transparent'}" >{{edge}}</span></td>
              </tr>
          </table>
        </div>
      </div>
    </div>
    </div>
    <br/>
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
        <template v-for="(chains, model) in result.models">
          <template v-for="(links, chain) in chains">
            <tr :key="model+chain">
              <td>{{model}}</td>
              <td>{{chain.slice(0,1)}}</td>
              <td>{{chain.slice(1,2)}}</td>
              <td><button v-if="links.PDB" class="btn btn-secondary" v-on:click='open2D(links.PDB.PDB, links.parsed_bonds, `${chain.slice(0,1)} and ${chain.slice(1,2)} chains from model ${model}`)'>2D Display</button></td>
              <td><button v-if="links.PDB" class="btn btn-secondary" v-on:click='open3D(links.PDB.PDB, chain.slice(0,1), chain.slice(1,2))'>3D Display</button></td>
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
    result: '',
    display_graph: false,
    edge_colors: {
      backbone: '#666',
      HYDPHB: '#880000',
      ELCSTA: '#000088',
      HYBOND: '#008800'
    },
    s: undefined,
    info: '',
  }),
  mounted: function () {
    this.$http.get(this.$server_url + '/results/' + this.$route.params.job_id).then(function (response) {
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
      &select=chain:${chain2}&style=cartoon:color~yellow`)
    },
    parsepdb: function (pdb) {
      let residues = [];
      let bonds = [];
      let min_chain_idx = {};
      let offset = 0;
      pdb.split('\n').forEach((line) => {
        let chain;
        let aa;
        let index;
        if (line.slice(0, 4) === 'ATOM') {
          aa = line.substring(17, 21).trim();
          chain = line.substring(21, 22).trim();
          index = line.substring(22, 26).trim();
          if (!min_chain_idx[chain]) {
            min_chain_idx[chain] = parseInt(index);
          }
          let resid = chain + index;
          if (!residues.length || residues[residues.length - 1].id !== resid) {
            if (residues.length && residues[residues.length - 1].id[0] === chain) {
              bonds.push({
                id: bonds.length,
                label: chain + ' backbone',
                source: residues[residues.length - 1].id,
                target: resid,
                color: this.edge_colors.backbone,
                size: 0.1,
              });
            } else {
              offset = residues.length
            }
            residues.push({
              id: resid,
              label: `${resid}: ${aa}`,
              x: 2 * (parseInt(index) - min_chain_idx[chain]),
              y: offset,
              color: this.edge_colors.backbone,
              size: 0.1,
            });
          }
        }
      });
      return {'nodes': residues, 'edges': bonds}
    },
    extractParsedXML: function (xml_json) {
      let new_edges = [];
      JSON.parse(xml_json).BONDS.BOND.forEach((bond, index) => {
        new_edges.push({
          id: `M${index}`,
          label: bond.type._text,
          size: 1 / parseFloat(bond.dist._text),
          source: bond.RESIDUE[0].chain._text + bond.RESIDUE[0]._attributes.index,
          target: bond.RESIDUE[1].chain._text + bond.RESIDUE[1]._attributes.index,
          color: this.edge_colors[bond.type._text],
        })
      });
      return new_edges
    },
    close2D: function () {
      this.display_graph = false;
      this.s = undefined;
    },
    open2D: function (pdbFile, parsed_xml, label) {
      this.display_graph = label;
      if (!this.s) {
        // Instantiate sigma, use SetTimeout to give Vue a chance to load the container
        setTimeout(() => {
          this.s = new sigma({ // eslint-disable-line
            renderer: {
              container: document.getElementById('graph-container'),
              type: 'canvas'
            },
            settings: {
              drawLabels: true,
              maxNodeSize: 2,
              minEdgeSize: 0.2,
              maxEdgeSize: 2,
              enableEdgeHovering: true,
              edgeHoverSizeRatio: 4,
              edgeHoverExtremities: true,
              sideMargin: 5,
              singleHover: true,
            }
          })
        }, 100);
      }
      this.$http.get(pdbFile).then((response) => {
        response.text().then((pdb) => {
          let graph = this.parsepdb(pdb);
          graph.edges = graph.edges.concat(this.extractParsedXML(parsed_xml));
          this.s.graph.clear();
          this.s.graph.read(graph);
          this.s.refresh();
        });
      });
    },
  }
}
</script>

<style scoped>
  #graph-container {
    height: 400px;
    margin: auto;
  }
  #color-table {
    float: none;
    display: table-cell;
    vertical-align: bottom;
  }
</style>
