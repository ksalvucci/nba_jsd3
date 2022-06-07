<template>
  <div id="app">
    <ScatterPlot
      class="scatter-plot"
      v-if="dataset"
      chartId="poke1"
      title="Teams Overview"
      :dataset="dataset"
      attribX="rating"
      labelX="Total Rating"
      attribY="salaries"
      labelY="Total Spending (in $100,000)"
      attribColor="name"
      @item-selected="teamSelected"
      :width="400"
      :height="200"
    />
    <BarChart
      class="bar-chart"
      v-if="datasetSelected"
      chartId="poke2"
      :title="`Details about ${datasetSelected.name}`"
      :dataset="datasetSelected.entries"
      attribX="name"
      attribY="ratings"
      labelY="Ratings"
      :attribColor='selectedAttrib'
      :width="400"
      :height="200"
      @item-selected="playerSelected"
    />
    <!-- Bar Chart Remapping --> 
    <h1 v-else class="bar-chart-info">Select a team</h1>
    <div class='filter-box' v-if='datasetSelected'>
      <span>Select Color Mapping Attribute </span>
      <select v-model='selectedAttrib'>
        <option value='ratings'>Rating</option>
        <option value='salary'>Salary</option>
        <option value='height'>Height</option>
        <option value='weight'>Weight</option>
      </select>
    </div>

    <RadarChart
      class="radar-chart"
      v-if="barDatasetSelected"
      chartId="poke3"
      :title="`Details about ${barDatasetSelected.name}`"
      :dataset="barDatasetSelected.entries"
      :width="800"
      :height="500"
      @remove-player="removePlayer"
    />

    <!-- Radar Chart Filtering --> 
    <h1 v-else class="radar-chart-info">Select a player</h1>
    <div class="button-group" v-if="barDatasetSelected">
      <button class="btn" v-on:click="updateCurrentMethod('o')" :class="{ selected: selectedMetric == 'o' }">
        Offensive Stats
      </button>
      <button class="btn" v-on:click="updateCurrentMethod('d')" :class="{ selected: selectedMetric == 'd' }">
        Defensive Stats
      </button>
      <button class="btn" v-on:click="updateCurrentMethod('a')" :class="{ selected: selectedMetric == 'a' }">
        Atheltic Stats
      </button>
      <button class="btn" v-on:click="updateCurrentMethod('p')" :class="{ selected: selectedMetric == 'p' }">
        Playmaking Stats
      </button>
    </div>


  </div>
</template>

<script>
import * as d3 from "d3";

import ScatterPlot from "./components/ScatterPlot.vue";
import BarChart from "./components/BarChart.vue";
import RadarChart from "./components/RadarChart.vue";

export default {
  name: "App",
  components: {
    ScatterPlot,
    BarChart,
    RadarChart,
  },
  data() {
    return {
      data: null,
      dataset: null,
      datasetSelected: null,
      barDatasetSelected: null,
      playerDataset: null,
      selectedMetric: "o",
      selectedPlayers: [],
      selectedAttrib: 'ratings'
    };
  },
  created() {
    d3.csv("/data/nba2k20-full.csv", d3.autoType).then((data) => {
      let teams = [];
      for (var i = 0; i < data.length; i++) {
        teams[i] = data[i].team;
      }
      let unique = [];
      teams.forEach((t) => {
        if (!unique.includes(t)) {
          unique.push(t);
        }
      });
      teams = unique;

      let j = 0;
      let money = new Array(teams.length).fill(0);
      let salary = [];
      while (j < teams.length) {
        for (var k = 0; k < data.length; k++) {
          if (data[k].team == teams[j]) {
            money[j] = parseInt(data[k].salary.replace("$", "")) + money[j];
          }
        }
        j = j + 1;
      }
      salary = money;
      var i1 = 0;
      let ratings = new Array(teams.length).fill(0);
      while (i1 < teams.length) {
        for (var k1 = 0; k1 < data.length; k1++) {
          if (data[k1].team == teams[i1]) {
            ratings[i1] = Number(data[k1].rating) + ratings[i1];
          }
        }
        i1 = i1 + 1;
      }

      let table = new Array(teams.length).fill(0);
      for (var i2 = 0; i2 < teams.length; i2++) {
        table[i2] = {
          name: teams[i2],
          salary: salary[i2],
          ratings: ratings[i2],
        };
      }
      this.data = data;

      this.dataset = d3.map(table, (d) => {
        return Object.assign(d, {
          rating: d.ratings,
          salaries: d.salary
        });
      });
    });
  },
  methods: {
    teamSelected(p) {
      if (p == null) {
        this.datasetSelected = null;
        return;
      }
      let team = [];

      var thisteam = p.name;
      if (p.name == null) {
        thisteam = "Free Agents";
      }
      team = d3.filter(this.data, (d) => d.team == p.name);

      let entries = [];
      for (var i = 0; i < team.length; i++) {
        entries[i] = {
          name: team[i].full_name,
          ratings: team[i].rating,
          height: +team[i].height.slice(-4),
          weight: +team[i].weight.substring(0,3),
          salary: +team[i].salary.substring(1,).replace(/,/g,""),
        };
      }

      this.datasetSelected = {
        name: thisteam,
        entries: entries,
      };
    },
    playerSelected(p) {
      if (p == null) {
        this.barDatasetSelected = null;
        return;
      }

      var player = null;
      for (var i in this.data) {
        let item = this.data[i];
        if (item.full_name == p.name) {
          player = item;
        }
      }

      if (this.selectedPlayers.indexOf(player) > -1) {
        return;
      }

      this.selectedPlayers.push(player);
      this.updatePlayerData();
    },
    updatePlayerData() {
      // create the axis data

      var allData = [];
      for (let player of this.selectedPlayers) {
        if (this.selectedMetric == 'o') {
          allData.push(this.createOffensivePlayerData(player));
        }
        if (this.selectedMetric == 'd') {
          allData.push(this.createDefensivePlayerData(player));
        }
        if (this.selectedMetric == 'a') {
          allData.push(this.createAthleticPlayerData(player));
        }
        if (this.selectedMetric == 'p') {
          allData.push(this.createPlaymakingPlayerData(player));
        }
      }

      var name = '';
      if (this.selectedPlayers.length > 1) {
        name = this.selectedPlayers.length + ' selected players';
      } else {
        name = this.selectedPlayers[0].full_name;
      }

      this.barDatasetSelected = {
        name: name,
        entries: allData,
      };
    },
    createOffensivePlayerData(player) {
      let allData = [];
      allData.push({ axis: "Shot Close", value: player.shot_close, name: player.full_name });
      allData.push({ axis: "Mid-Range Shot", value: player.shot_mid, name: player.full_name });
      allData.push({ axis: "3-PT Shot", value: player.shot_3pt, name: player.full_name });
      allData.push({ axis: "Shot I.Q.", value: player.shot_iq, name: player.full_name });
      allData.push({ axis: "Offensive Consistency", value: player.offensive_consistency, name: player.full_name});
      allData.push({ axis: "Driving Dunk", value: player.driving_dunk, name: player.full_name });
      allData.push({ axis: "Standing Dunk", value: player.standing_dunk, name: player.full_name });
      allData.push({ axis: "Driving Layup", value: player.driving_layup, name: player.full_name });
      return allData;
    },
    createDefensivePlayerData(player) {
      let allData = [];
      allData.push({ axis: "Interior Defense", value: player.interior_defense, name: player.full_name });
      allData.push({ axis: "Perimeter Defense", value: player.perimeter_defense, name: player.full_name });
      allData.push({ axis: "Help Defense Iq", value: player.help_defense_iq, name: player.full_name });
      allData.push({ axis: "Lateral Quickness", value: player.lateral_quickness, name: player.full_name });
      allData.push({ axis: "Pass Perception", value: player.pass_perception, name: player.full_name});
      allData.push({ axis: "Reaction Time", value: player.reaction_time, name: player.full_name });
      allData.push({ axis: "Steal", value: player.steal, name: player.full_name });
      allData.push({ axis: "Block", value: player.block, name: player.full_name });
      return allData;
    },
    createAthleticPlayerData(player) {
      let allData = [];
      allData.push({ axis: "Speed", value: player.speed, name: player.full_name });
      allData.push({ axis: "Acceleration", value: player.acceleration, name: player.full_name });
      allData.push({ axis: "Vertical", value: player.vertical, name: player.full_name });
      allData.push({ axis: "Strength", value: player.strength, name: player.full_name });
      allData.push({ axis: "Stamina", value: player.stamina, name: player.full_name});
      allData.push({ axis: "Hustle", value: player.hustle, name: player.full_name });
      allData.push({ axis: "Hands", value: player.hands, name: player.full_name });
      allData.push({ axis: "Durability", value: player.overall_durability, name: player.full_name });
      return allData;
    },
    createPlaymakingPlayerData(player) {
      let allData = [];
      allData.push({ axis: "Ball Handle", value: player.ball_handle, name: player.full_name });
      allData.push({ axis: "Speed w/ Ball", value: player.speed_with_ball, name: player.full_name });
      allData.push({ axis: "Pass Accuracy", value: player.passing_accuracy, name: player.full_name });
      allData.push({ axis: "Pass Vision", value: player.passing_vision, name: player.full_name });
      allData.push({ axis: "Passing I.Q.", value: player.passing_iq, name: player.full_name});
      allData.push({ axis: "Flashy Pass", value: player.flashy_pass_t, name: player.full_name });
      allData.push({ axis: "Alley oop Pass", value: player.alley_oop_pass_t, name: player.full_name });
      allData.push({ axis: "Open Pass", value: player.dish_to_open_man_t, name: player.full_name });
      return allData;
    },
    updateCurrentMethod(i) {
      this.selectedMetric = i;
      this.updatePlayerData();
    },
    removePlayer(p) {
      console.log("player: ", p);
      var index = -1;
      for (let ply in this.selectedPlayers) {
        this.selectedPlayers[ply].name == p[0].name;
        index = ply;
      }

      if (index > -1) {
        this.selectedPlayers.splice(index, 1);
      }

      if (this.selectedPlayers.length == 0) {
        this.barDatasetSelected = null;
        return;
      }

      this.updatePlayerData();
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: hsl(210, 29%, 24%);
  display: grid;
  grid-template-rows: 5vh 60vh auto;
  grid-template-columns: 1fr 10fr 10fr 1fr;
  grid-template-areas:
    ". . . ."
    ". overview detail ."
    ". rad but .";
}

.scatter-plot {
  grid-area: overview;
}

.bar-chart {
  grid-area: detail;
}

.radar-chart {
  grid-area: rad;
}

.bar-chart-info {
  grid-area: detail;
  color: rgb(214, 214, 214);
  margin: auto;
}

.radar-chart-info {
  margin-top: 64px;
  grid-area: rad;
  color: rgb(214, 214, 214);
}

.button-group {
  margin-top: 75px;
  grid-area: but;
}

.filter-box {
  grid-area: but;
  margin-left: 150px;
}

.btn {
  display: block;
  background: none;
  margin: 10px;
  color: black;
  border-color: black;
}

.selected {
  background: red;
  color: white;
}
</style>
