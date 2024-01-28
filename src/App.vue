<template>
  <div class="game-container">
    <h1>NBA Games on<br> {{ searchDate }}</h1>
    <div style="display: flex; justify-content: space-between; width: 60%; margin: auto;">
      <button @click="goToPreviousDate">Previous Day</button>
      <button @click="goToNextDate">Next Day</button>
    </div>
    <div v-if="isFetchingData" class="loading">Loading...</div>
    <ul v-else class="game-list">
      <template v-for="game in games" :key="game.id" >
        <li class="game-item" @click="game.isSpoiled = !game.isSpoiled">
          <span>{{ game.home_team.abbreviation }}</span> vs <span>{{ game.visitor_team.abbreviation }}</span><br>
          <small>{{ game.status }}</small>
          <strong v-if="game.isSpoiled"><br>{{ game.home_team_score }}:{{ game.visitor_team_score }}</strong><br>
        </li> 
      </template>
    </ul>
  </div>
</template>

<script>
import moment from 'moment-timezone';

export default {
  data() {
    return {
      games: [],
      searchDate: '2024-01-27', // Example date in YYYY-MM-DD format
      isFetchingData: false
    }
  },
  async mounted() {
    console.clear()
    this.searchDate = await this.getCurrentDate ()
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
</style>
