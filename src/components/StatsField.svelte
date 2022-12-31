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
            team: i.statistics[0].team.logo,
            minutes: i.statistics[0].games.minutes,
            assists: i.statistics[0].goals.assists,
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
      <span class="player">Player</span>
      <div class="figures">
        <span class="additional">Mins</span>
        <span class="additional">Assists</span>
        <span>Goals</span>
        <span>Pens</span>
      </div>
    </div>
  {#each stats as player}
    <div class="player">
      <span class="player-group">
        <img src="{player.team}" alt="teamLogo">
        <span class="name">{player.player}</span>
      </span>
      <div class="figures">
        <span class="additional">{player.minutes}</span>
        <span class="additional">{player.assists === null ? '-' : player.assists}</span>
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
   div.statsHeader {
     border-radius: 5px 5px 0 0;
     background-color: #59656F;
   }
   div.player {
    background-color: #C4D4CA;
   }
   span.player-group {
    display: grid;
    place-items: center;
    grid-template-columns: repeat(2, auto);
    gap: 1em;
   }
  div.statsHeader, div.player {
    width: 100%;
    height: 2em;
    display: grid;
    grid-template-columns: 50% 50%;
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
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1em;
    text-align: right;
  }
  @media screen and (min-width: 640px) {
    div.stats {
      width: 60%;
    }
  }
  @media screen and (max-width: 760px) {
    div.figures {
      grid-template-columns: repeat(2, 1fr);
    }
    span.additional {
      display: none;
    }
  }
</style>

