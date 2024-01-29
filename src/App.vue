<template>
  <div class="game-container">
    <h1>NBA Games on<br> {{ searchDate }}</h1>
    <div style="display: grid; justify-content: space-around; width: 80%; margin: auto;grid-template-columns: 45% 45%;">
      <button @click="goToPreviousDate()"  style="margin-bottom: 10px;">Previous Day</button>
      <button @click="goToNextDate()" style="margin-bottom: 10px;">Next Day</button>
      <input type="date" v-model="searchDate">
      <button @click="showScore()">Show Scores</button>
    </div>
    <div v-if="isFetchingData" class="loading">Loading...</div>
    <ul v-else class="game-list">
      <template v-for="game in games" :key="game.id" >
       <li class="game-item" @click="game.isSpoiled = !game.isSpoiled" data-aos="fade-up" :style="getBackgroundColor(game.home_team.abbreviation)">
        <div class="list-top">
          <div class="team-logo" :style="getTeamLogoPosition(game.home_team.abbreviation)"></div>
          <div class="team-data">
            <p>{{ game.home_team.abbreviation }} <br>vs<br>{{ game.visitor_team.abbreviation }}</p>
          </div>
          <div class="team-logo" :style="getTeamLogoPosition(game.visitor_team.abbreviation)"></div>
        </div>
        <div class="list-bottom">
          <div>
            <span v-if="game.isSpoiled">{{ game.home_team_score }}</span>
          </div>
          <div class="game-status">
            <span v-html="getStatus(game)"> </span>
            <br><strong v-if="isGameHot(game)">🔥</strong>
          </div>
          <div>
            <span v-if="game.isSpoiled">{{ game.visitor_team_score }}</span>
          </div>
        </div>
          <!-- <div class="game-data">
            <div class="team-logo" :style="getTeamLogoPosition(game.home_team.abbreviation)"></div>{{ game.home_team.abbreviation }} vs {{ game.visitor_team.abbreviation }}
            <div class="team-logo" :style="getTeamLogoPosition(game.visitor_team.abbreviation)"></div>
          </div>
          <small>{{ game.status }} <strong v-if="isGameHot(game)">🔥</strong></small>
          <strong v-if="game.isSpoiled"><br>{{ game.home_team_score }}:{{ game.visitor_team_score }}</strong><br> -->
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
        this.sortGames();
        console.log(this.games);
      } catch (error) {
        console.error(error);
      } finally {
        this.isFetchingData = false;
      }
    },

    getCurrentDate() {
      const now = moment().tz('Asia/Tokyo');
      now.subtract(1, 'days');
      // if (now.hour() > 6)
      // if (now.hour() > 6) {
      //   console.log('here')
      //   now.subtract(2, 'days');
      // }
      return now.format('YYYY-MM-DD');
    },

    goToPreviousDate() {
      this.adjustDate(-1);
      // this.fetchGames();
    },
    goToNextDate() {
      this.adjustDate(1);
      // this.fetchGames();
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
      if(scoreDifference < 10) return true
      return false
    },

    getStatus(game){
      if(game.time) return game.status
      let tempStatus = game.status
      tempStatus = tempStatus.substring(11)
      tempStatus = tempStatus.slice(0, -4);

      let firstTwoLetters = tempStatus.substring(0, 2);
      firstTwoLetters = parseInt(firstTwoLetters);
      let changedHours = firstTwoLetters + 9
      if(changedHours > 24) changedHours =changedHours-24
      return `Start from <br> ${changedHours}${tempStatus.substring(2)}`
    },

    sortGames(){
      this.games = this.games.sort((a, b) => a.id - b.id);
    },

    getBackgroundColor(abbreviation) {
      
      const mapData = [
        { team: 'GSW', color: '#b4ac66', fontColor: 'white'},
        { team: 'DEN', color: '#79a1b3', fontColor: 'white'},
        { team: 'DAL', color: '#004381', fontColor: 'white'},
        { team: 'CHI', color: '#990000', fontColor: 'white'},
        { team: 'ATL', color: '#990000', fontColor: 'white'},
        { team: 'BOS', color: '#005538', fontColor: 'white'},

        { team: 'LAC', color: '#990000', fontColor: 'white'},
        { team: 'MIN', color: '#001d3b', fontColor: 'white'},
        { team: 'HOU', color: '#990000', fontColor: 'white'},
        { team: 'CLE', color: '#75141a', fontColor: 'white'},
        { team: 'CHA', color: '#3b1a52', fontColor: 'white'},
        { team: 'BKN', color: '#000000', fontColor: 'white'},

        { team: 'LAL', color: '#b4ac66', fontColor: 'white'},
        { team: 'OKC', color: '#004381', fontColor: 'white'},
        { team: 'MEM', color: '#1a1c55', fontColor: 'white'},
        { team: 'DET', color: '#990000', fontColor: 'white'},
        { team: 'MIA', color: '#75141a', fontColor: 'white'},
        { team: 'NYK', color: '#9b3815', fontColor: 'white'},

        { team: 'PHX', color: '#3b1a52', fontColor: 'white'},
        { team: 'POR', color: '#960000', fontColor: 'white'},
        { team: 'NOP', color: '#0b0a32', fontColor: 'white'},
        { team: 'IND', color: '#986607', fontColor: 'white'},
        { team: 'ORL', color: '#004281', fontColor: 'white'},
        { team: 'PHI', color: '#0000b2', fontColor: 'white'},

        { team: 'SAC', color: '#3b1a52', fontColor: 'white'},
        { team: 'UTA', color: '#0b0a32', fontColor: 'white'},
        { team: 'SAS', color: '#242424', fontColor: 'white'},
        { team: 'MIL', color: '#003420', fontColor: 'white'},
        { team: 'WAS', color: '#0b0a32', fontColor: 'white'},
        { team: 'TOR', color: '#75141a', fontColor: 'white'},
      ];

      let color, fontColor;
      const team = mapData.find(teamData => {
        if (teamData.team === abbreviation) {
          color = teamData.color;
          fontColor = teamData.fontColor;
          return true; 
        }
        return false; 
      });

      if (team) {
        return `background: ${color}; color: ${fontColor}`; 
      } else {
        return 
      }
    },


  },
  watch: {
    // Watch for changes in searchDate
    searchDate(newDate, oldDate) {
      // Prevent fetching on initial setup or if the date hasn't changed
      if (newDate !== oldDate) {
        this.fetchGames();
      }
    }
  },
}
</script>

<style>
  html,body{
    background: #194045;
    font-family: 'Teko', sans-serif;
    /* color: forestgreen; */
    color: white;
  }

  h1{
    margin-bottom: 5px;
  }
  .game-container {
    text-align: center;
    max-width: 600px;
    margin: auto;
    padding: 5px;
  }

  .loading {
    font-size: 20px;
    color: #666;
  }

  .game-list {
    display: grid;
    grid-template-columns: 48% 48%;
    justify-content: space-between;

    list-style-type: none;
    padding: 0; 
    
  }

  

  

  .game-item {
    background-color: #f2f2f2;
    margin: 10px 0;
    padding: 10px 15px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.2);
  }

  

  .game-item .list-top{
    display: grid;
    grid-template-columns: 35% 30% 35%;
    justify-content: space-between;
    align-items: center;

    list-style-type: none;
    padding: 0;
  }

  .game-item .list-top p{
    font-size: 1.2em;
    line-height: 1;
    margin: 0;
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
    
    margin: auto;
    position: relative;

  }

  .team-logo::after{
    content: "";
    display: block;
    position: absolute;
    width: 100%;
    height: 1.5px;
    left: 50%;
    transform: translateX(-50%);
    background: white;
    top: calc(100% + 7.5px);
  }

  .game-item .list-bottom{
    /* margin-top: 15px; */
    display: grid;
    grid-template-columns: 12.5% 70% 12.5%;
    justify-content: space-between;
    align-items: center;

    list-style-type: none;
    padding: 0;
    line-height: 1;
    /* width: 85%; */
    margin: 15px auto 0px;

    font-size: 1.25em;
  }

  .game-item .list-bottom span{
    line-height: 1;
    margin: 0;
    /* color: #000; */
  }

  .game-item .list-bottom .game-status{
    font-size: 1.1em;
    text-align: center;
  }
</style>
