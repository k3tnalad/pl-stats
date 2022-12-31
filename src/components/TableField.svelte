<script>
  import { fly, fade } from 'svelte/transition';
  import '/src/app.css';

  const standingsEndpoint =  "https://api-football-v1.p.rapidapi.com/v3/standings?season=2022&league=39";
  const options = {
    method: 'GET',
    headers: {
      'X-RapidAPI-Key': '9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a',
      'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
    }
  };

  const nameShortener = (string) => {
    if (string === 'Nottingham Forest') {
      return 'Nottm Forest'
    } else if (string === 'Manchester City') {
      return 'Man City'
    } else if (string === 'Manchester United') {
      return 'Man United'
    } else if (string === 'Crystal Palace'){
      return 'C. Palace';
    } else {
      return string;
    }
  }

  let standings = JSON.parse(localStorage.getItem('teams')) || []; 

  async function getTableData() {
    if (standings.length > 1) {
      return
    } else { 
      const tableReq= await fetch(standingsEndpoint, options);
      if (tableReq.ok) {
        const tableData = await tableReq.json();
        standings = tableData.response[0].league.standings[0];
        localStorage.setItem('teams', JSON.stringify(standings));
      }
    }
    standings = JSON.parse(localStorage.getItem('teams'))
  }

  getTableData();
</script>

<div in:fly="{{x: -200, duration: 400}}" out:fade="{{ duration: 100}}" class="table">
    <div class="header">
      <span id="text-container" class="position-index"><p>#</p></span>
      <span class="team">Team</span>
      <span id="text-container"><p>+/-</p></span>
      <span id="text-container"><p>Pts</p></span>
    </div>
    {#each standings as team}
      <div class="position">
        <span id="text-container" class="positon-index"><p>{team.rank}</p></span>
        <span class="team">
          <img src={team.team.logo} alt="teamLogo">
          <span>{nameShortener(team.team.name)}</span></span>
        <span id="text-container"><p>{team.all.goals.for}/{team.all.goals.against}</p></span>
        <span id="text-container"><p>{team.points}</p></span>
      </div>
    {/each}
</div>

<style>
    div.table {
      width: 95%;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      box-shadow: rgba(0, 0, 0, 0.25) 0px 14px 28px, rgba(0, 0, 0, 0.22) 0px 10px 10px;
      border-radius: 5px;
   }

   @media screen and (min-width: 640px) {
    div.table {
      width: 60%;
    }
   }

  div.position, div.header {
    width: 100%;
    height: 2em;
    color: #040F16;
    display: grid;
    grid-template-columns: 10% auto 15% 15%;
    background-color: #72727E;
    gap: 1em; 
    border-bottom: 2px solid black;
  }

  div.position:last-of-type {
    border: none;
    border-radius: 0px 0px 5px 5px;
  }

  div.position > span:first-of-type,
  div.header > span:first-of-type {
    border-right: 2px solid black;
    text-align: right;
    padding-right: 10px;
  } 

  div.header {
    border-radius: 5px 5px 0px 0px;
    background-color: #4E4E56;
    color: #040F16;
  }

  span.team {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    gap: 0.4em;
  }
  span#text-container {
    display: grid;
    place-items: center;
  }
</style>