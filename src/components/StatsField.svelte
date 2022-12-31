<script>
import { fly, fade } from 'svelte/transition';
import '/src/app.css';

const statisticsEndpoint = 'https://api-football-v1.p.rapidapi.com/v3/players/topscorers?league=39&season=2022';
  const options = {
    method: 'GET',
    headers: {
      'X-RapidAPI-Key': '9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a',
      'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
    }
};
let stats = JSON.parse(localStorage.getItem('stats')) || [];

async function getStatsData() {
    if (stats.length > 1) {
      return
    } else { 
      const statisticsReq = await fetch(statisticsEndpoint, options)
      if (statisticsReq.ok) {
        const statisticsData = await statisticsReq.json();
        stats = statisticsData.response.map(i => {
          return {
            player: i.player.name,
            goals: i.statistics[0].goals.total,
            pens: i.statistics[0].penalty.scored,
          }
        });
        localStorage.setItem('stats', JSON.stringify(stats));
      }
    }
    stats = JSON.parse(localStorage.getItem('stats'))
  } 

  getStatsData();


</script>

<div in:fly="{{x: -200, duration: 400}}" out:fade="{{ duration: 100}}" class="stats">
    <div class="statsHeader">
      <span>Player</span>
      <div class="figures">
        <span>Goals</span>
        <span>Pens</span>
      </div>
    </div>
  {#each stats as player}
    <div class="player">
      <span>{player.player}</span>
      <div class="figures">
        <span>{player.goals}</span>
        <span>{player.pens}</span>
      </div>
    </div>
  {/each}
</div>

<style>
div.stats {
    width: 95%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    box-shadow: rgba(0, 0, 0, 0.25) 0px 14px 28px, rgba(0, 0, 0, 0.22) 0px 10px 10px;
  }
  @media screen and (min-width: 640px) {
    div.stats {
      width: 60%;
    }
   }

   div.statsHeader {
     border-radius: 5px 5px 0 0;
     background-color: #4E4E56;
   }
   div.player {
    background-color: #72727E;
   }
  div.statsHeader, div.player {
    width: 100%;
    height: 2em;
    display: grid;
    grid-template-columns: 70% 30%;
    place-items: center left;
    padding: 0 1em;
    border-bottom: solid 2px black;
  }

  div.player:last-of-type {
    border: none;
    border-radius: 0 0 5px 5px;
  }

  div.figures {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
</style>

