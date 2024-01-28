<template>
  <div class="game-container">
    <h1>NBA Games on<br> {{ searchDate }}</h1>
    <div style="display: flex; justify-content: space-between; width: 100%; margin: auto;">
      <button @click="goToPreviousDate()">Previous Day</button>
      <button @click="showScore()">Show everything</button>
      <button @click="goToNextDate()">Next Day</button>
    </div>
    <div v-if="isFetchingData" class="loading">Loading...</div>
    <ul v-else class="game-list">
      <template v-for="game in games" :key="game.id" >
       <li class="game-item" @click="game.isSpoiled = !game.isSpoiled" data-aos="fade-up">
          <div class="game-data">
            <div class="team-logo" :style="getTeamLogoPosition(game.home_team.abbreviation)"></div>{{ game.home_team.abbreviation }} vs {{ game.visitor_team.abbreviation }}
            <div class="team-logo" :style="getTeamLogoPosition(game.visitor_team.abbreviation)"></div>
          </div>
          <small>{{ game.status }} <strong v-if="isGameHot(game)">🔥</strong></small>
          <strong v-if="game.isSpoiled"><br>{{ game.home_team_score }}:{{ game.visitor_team_score }}</strong><br>
        </li> 
      </template>
    </ul>
  </div>
</template>

<script>
import moment from 'moment-timezone';
import AOS from 'aos'
import 'aos/dist/aos.css'

export default {
  data() {
    return {
      games: [],
      searchDate: '',
      isFetchingData: false
    }
  },
  async mounted() {
    console.clear()
    AOS.init();

    this.searchDate = await this.getCurrentDate ()
    // this.searchDate = '2022-11-7'
    this.fetchGames();
  },
  methods: {
    async fetchGames() {
      this.isFetchingData = true;
      
      const URL = `https://www.balldontlie.io/api/v1/games?start_date=${this.searchDate}&end_date=${this.searchDate}`;
      try {
        const res = await fetch(URL);
        const json = await res.json();
        this.games = json.data;
        console.log(this.games);
      } catch (error) {
        console.error(error);
      } finally {
        this.isFetchingData = false;
      }
    },

    getCurrentDate() {
      const now = moment().tz('Asia/Tokyo');
      if (now.hour() > 6) {
        now.subtract(1, 'days');
      }
      return now.format('YYYY-MM-DD');
    },

    goToPreviousDate() {
      this.adjustDate(-1);
      this.fetchGames();
    },
    goToNextDate() {
      this.adjustDate(1);
      this.fetchGames();
    },
    adjustDate(days) {
      let currentDate = moment(this.searchDate).tz('Asia/Tokyo');
      currentDate.add(days, 'days');
      this.searchDate = currentDate.format('YYYY-MM-DD');
    },

    getTeamLogoPosition(abbreviation) {
      
      const mapData = [
        { team: 'GSW', position: { x: 0, y: 0 }},
        { team: 'DEN', position: { x: 1, y: 0 }},
        { team: 'DAL', position: { x: 2, y: 0 }},
        { team: 'CHI', position: { x: 3, y: 0 }},
        { team: 'ATL', position: { x: 4, y: 0 }},
        { team: 'BOS', position: { x: 5, y: 0 }},

        { team: 'LAC', position: { x: 0, y: 1 }},
        { team: 'MIN', position: { x: 1, y: 1 }},
        { team: 'HOU', position: { x: 2, y: 1 }},
        { team: 'CLE', position: { x: 3, y: 1 }},
        { team: 'CHA', position: { x: 4, y: 1 }},
        { team: 'BKN', position: { x: 5, y: 1 }},

        { team: 'LAL', position: { x: 0, y: 2 }},
        { team: 'OKC', position: { x: 1, y: 2 }},
        { team: 'MEM', position: { x: 2, y: 2 }},
        { team: 'DET', position: { x: 3, y: 2 }},
        { team: 'MIA', position: { x: 4, y: 2 }},
        { team: 'NYK', position: { x: 5, y: 2 }},

        { team: 'PHX', position: { x: 0, y: 3 }},
        { team: 'POR', position: { x: 1, y: 3 }},
        { team: 'NOP', position: { x: 2, y: 3 }},
        { team: 'IND', position: { x: 3, y: 3 }},
        { team: 'ORL', position: { x: 4, y: 3 }},
        { team: 'PHI', position: { x: 5, y: 3 }},

        { team: 'SAC', position: { x: 0, y: 4 }},
        { team: 'UTA', position: { x: 1, y: 4 }},
        { team: 'SAS', position: { x: 2, y: 4 }},
        { team: 'MIL', position: { x: 3, y: 4 }},
        { team: 'WAS', position: { x: 4, y: 4 }},
        { team: 'TOR', position: { x: 5, y: 4 }},
      ];

      let tempXcode, tempYcode;
      const team = mapData.find(teamData => {
        if (teamData.team === abbreviation) {
          tempXcode = teamData.position.x;
          tempYcode = teamData.position.y;
          return true; 
        }
        return false; 
      });
      const margin = -54
      const calCultedXcode = (margin * tempXcode) -2
      const calCultedYcode = (margin * tempYcode) -2

      if (team) {
        return `background-position: ${calCultedXcode}px ${calCultedYcode}px`; 
      } else {
        return 'background-position: 49px 40px';
      }
    },

    showScore(){
      for (let game of this.games) {
        game.isSpoiled = true;
      }
    },

    isGameHot(game){
      if(!game.time) return false
      const scoreDifference = Math.abs(game.home_team_score - game.visitor_team_score);
      console.log(scoreDifference);
      if(scoreDifference < 10) return true
      return false
    },


  }
}
</script>

<style>
  .game-container {
    text-align: center;
    max-width: 600px;
    margin: auto;
    padding: 20px;
  }

  .loading {
    font-size: 20px;
    color: #666;
  }

  .game-list {
    list-style-type: none;
    padding: 0;
  }

  .game-item {
    background-color: #f2f2f2;
    margin: 10px 0;
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .game-item span {
    font-weight: bold;
  }

  .game-data{
    display: flex;
    align-items: center;
    justify-content: center;

    margin: 0px 0px 10px;
  }

  .team-logo{
    width: 50px;
    height: 50px;
    /* height: 100px; */
    border-radius: 50%;
    background-size: 324px 270.5px; /* This scales down the image */
    /* background-position: -2px -2px; */
    background-image: url('./assets/team-logo.png');
    background-repeat: no-repeat; 
    
    margin: auto 15px;

  }
</style>
