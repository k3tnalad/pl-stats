<script>
  import { fade, fly } from 'svelte/transition';
  import '/src/app.scss';
  const standingsEndpoint =  "https://api-football-v1.p.rapidapi.com/v3/standings?season=2022&league=39";
  const fixturesEndpoint = "https://api-football-v1.p.rapidapi.com/v3/fixtures?league=39&season=2022&last=10";
  const statisticsEndpoint = 'https://api-football-v1.p.rapidapi.com/v3/players/topscorers?league=39&season=2022';
  const options = {
    method: 'GET',
    headers: {
      'X-RapidAPI-Key': '9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a',
      'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
    }
  };

  let lastClicked = 'Table';
  let tableVisible = true;
  let fixturesVisible = false;
  let statsVisible = false;

  const navHandler = e => {
    let section = e.target.textContent; 
    if (e.target.textContent === section && lastClicked === section) {
      return;
    }  else if (e.target.textContent === 'Table') {
      tableVisible = true;
      fixturesVisible = false;
      lastClicked = section;
    } else if (e.target.textContent === 'Fixtures') {
      fixturesVisible = true;
      tableVisible = false;
      statsVisible = false;
      lastClicked = section;
    } else if (e.target.textContent === 'Statistics') {
      fixturesVisible = false;
      tableVisible = false;
      statsVisible = true;
      lastClicked = section;
    }
  }


  let standings = JSON.parse(localStorage.getItem('teams')) || []; 
  let fixtures = JSON.parse(localStorage.getItem('fixtures')) || [];
  let stats = JSON.parse(localStorage.getItem('stats')) || [];

  async function getLeagueData() {
    if (standings.length > 1) {
      return
    } else { 
      const [tableReq, fixturesReq, statisticsReq] = await Promise.all([fetch(standingsEndpoint, options), fetch(fixturesEndpoint, options), fetch(statisticsEndpoint, options)])
      if (tableReq.ok && fixturesReq.ok, statisticsReq.ok) {
        // standings
        const tableData = await tableReq.json();
        standings = tableData.response[0].league.standings[0];
        localStorage.setItem('teams', JSON.stringify(standings));
        //fixtures
        const fixturesData = await fixturesReq.json();
        fixtures = fixturesData.response;
        localStorage.setItem('fixtures', JSON.stringify(fixtures));
        //statistics
        const statisticsData = await statisticsReq.json();
        stats = statisticsData.response.map(i => {
          return {
            player: i.player.name,
            goals: i.statistics[0].goals.total,
            pens: i.statistics[0].penalty.scored,
          }
        });
        console.log(stats)
        localStorage.setItem('stats', JSON.stringify(stats));
      }
    }
    standings = JSON.parse(localStorage.getItem('teams'))
    fixtures = JSON.parse(localStorage.getItem('fixtures'))
    stats = JSON.parse(localStorage.getItem('stats'))
    console.log(stats);
  } 

  getLeagueData();

</script>

<main>
  <header>
    <h1>English Premier League</h1>
  </header>
  <nav>
    <div class="wrapper">
      <button on:click={navHandler}>Fixtures</button>
      <button on:click={navHandler}>Table</button>
      <button on:click={navHandler}>Statistics</button>
    </div>
  </nav>
  {#if tableVisible}  
    <div transition:fly="{{delay: 20, duration: 200}}" class="table">
      <div class="header">
        <span>Pos</span>
        <span>Team</span>
        <span>+/-</span>
        <span>Pts</span>
      </div>
      {#each standings as team}
        <div class="position" dataset={team.rank}>
          <span>{team.rank}</span>
          <span>{team.team.name}</span>
          <span>{team.all.goals.for}/{team.all.goals.against}</span>
          <span>{team.points}</span>
        </div>
      {/each}
    </div>
  {/if}
  {#if fixturesVisible}  
    <div transition:fly="{{delay: 20, duration: 200}}" class="fixtures">
      {#each fixtures as fixture}
        <div class="line" id={fixture.fixture.id}>
          <span class="homeTeam">{fixture.teams.home.name}</span>
          <section class="info">
            <!-- <span class="status">{fixture.fixture.status.short}</span> -->
            <span class="score">{fixture.goals.home} - {fixture.goals.away}</span>
          </section>
          <span class="awayTeam">{fixture.teams.away.name}</span>
        </div>
        {/each}
    </div>
  {/if}
  {#if statsVisible}
    <div transition:fly="{{delay: 20, duration: 200}}" class="stats">
      <div class="statsHeader">
        <span>Player</span>
        <span>Goals</span>
        <span>Of penalties</span>
      </div>
    {#each stats as player}
      <div class="player">
        <span>{player.player}</span>
        <span>{player.goals}</span>
        <span>{player.pens}</span>
      </div>
    {/each}
    </div>
  {/if}
</main>

<style>
  main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    flex-direction: column;
    gap: 3rem;  
    padding-bottom: 3rem;
  }
  header {
    padding: 2em 0;
  }

  div.table, div.fixtures, div.stats {
    width: 90%;
    border: 1px #8B2635 solid;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    gap: 2px;
  }
  div.position, div.header {
    width: 100%;
    color: #F2F4F3;
    display: grid;
    grid-template-columns: 10% auto 20% 10%;
    background-color: #8B2635;
    gap: 1em;
    padding: 0 .5em; 
  }

  div.header {
    background-color: #50151e;
  }

  div.statsHeader, div.player {
    background-color: #50151e;
    width: 100%;
    height: 50px;
    display: grid;
    grid-template-columns: 2fr 1fr 1fr;
  }
  div.player {
    background-color: #8B2635;
  }


  div.line {
    width: 100%;
    height: 50px;
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    font-size: 1em;
    background: #8B2635;
  }

  span, section.info {
    display: grid;
    place-items: center;
  }

  section.info {
    background: #50151e;
    padding: 0 .6em;
  }

  button {
    outline: none;
    background-color: #8B2635;
    color: #F2F4F3;
    border: none;
    padding: 1rem;
    font-size: larger;
    border-radius: 2px;
    cursor: pointer;
    box-shadow: 0px -2px 4px 2px rgba(0,0,0,0.6);
  }
</style>

