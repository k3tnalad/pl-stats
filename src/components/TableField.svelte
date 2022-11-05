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

<div in:fly="{{duration: 200}}" out:fade="{{ duration: 100}}" class="table">
    <div class="header">
      <span>#</span>
      <span class="team">Team</span>
      <span>+/-</span>
      <span>Pts</span>
    </div>
    {#each standings as team}
      <div class="position" dataset={team.rank}>
        <span>{team.rank}</span>
        <span class="team">
          <img src={team.team.logo} alt="teamLogo">
          <span>{nameShortener(team.team.name)}</span></span>
        <span>{team.all.goals.for}/{team.all.goals.against}</span>
        <span>{team.points}</span>
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
   }

   @media screen and (min-width: 640px) {
    div.table {
      width: 60%;
    }
   }

  div.position, div.header {
    width: 100%;
    color: #F2F4F3;
    display: grid;
    grid-template-columns: 10% auto 15% 15%;
    background-color: #8B2635;
    gap: 1em;
    padding: .5em .5em; 
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
    background-color: #50151e;
    color: #494e4a;
  }

  span.team {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    gap: 0.4em;
  }
</style>