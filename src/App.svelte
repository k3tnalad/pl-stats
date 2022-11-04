<script>
    import { prevent_default } from 'svelte/internal';
  import { fade, fly } from 'svelte/transition';
  import '/src/app.css';
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

  const HEADER_TITLE = 'premier league lobby';
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

  const nameShortener = (string) => {
    if (string === 'Nottingham Forest') {
      return 'Nottm Forest'
    } else if (string === 'Manchester City') {
      return 'Man City'
    } else if (string === 'Manchester United') {
      return 'Man United'
    } else {
      return string;
    }
  }


  let standings = JSON.parse(localStorage.getItem('teams')) || []; 
  let fixtures = JSON.parse(localStorage.getItem('fixtures')) || [];
  let stats = JSON.parse(localStorage.getItem('stats')) || [];

  console.log(fixtures)

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
    <h1>{HEADER_TITLE.toUpperCase()}</h1>
  </header>
  <nav>
    <div class="wrapper">
      <a href="/" on:click|preventDefault={navHandler}>Fixtures</a>
      <a href="/" on:click|preventDefault={navHandler}>Table</a>
      <a href="/" on:click|preventDefault={navHandler}>Statistics</a>
    </div>
  </nav>
  {#if tableVisible}  
    <div transition:fly="{{delay: 20, duration: 200}}" class="table">
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
  {/if}
  {#if fixturesVisible}  
    <div transition:fly="{{delay: 20, duration: 200}}" class="fixtures">
      {#each fixtures as fixture}
        <div class="line" id={fixture.fixture.id}>
          <span class="homeTeam">
            <img src="{fixture.teams.home.logo}" alt="teamLogo">
            <span>{nameShortener(fixture.teams.home.name)}</span>
          </span>
          <span class="score">{fixture.goals.home} - {fixture.goals.away}</span>
          <span class="awayTeam">
            <span>{nameShortener(fixture.teams.away.name)}</span>
            <img src="{fixture.teams.away.logo}" alt="teamLogo">
          </span>
        </div>
        {/each}
    </div>
  {/if}
  {#if statsVisible}
    <div transition:fly="{{delay: 20, duration: 200}}" class="stats">
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
    gap: 2rem;  
    padding-bottom: 3rem;
  }
  header {
    padding: 2em 0 1em 0;
    font-family: 'Crimson Text', serif;
  }

  /* general */

  div.table, div.fixtures, div.stats {
    width: 95%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
  }

  /* standings/table */

  div.position, div.header {
    width: 100%;
    color: #F2F4F3;
    display: grid;
    grid-template-columns: 10% auto 15% 15%;
    background-color: #8B2635;
    gap: 1em;
    padding: 0 .5em; 
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
    padding-right: 5px;
  } 

  div.header {
    border-radius: 5px 5px 0px 0px;
    background-color: #50151e;
    color: #494e4a;
  }

  /* stats */

  div.statsHeader, div.player {
    background-color: #50151e;
    width: 100%;
    height: 2em;
    display: grid;
    grid-template-columns: 70% 30%;
    place-items: center left;
    padding: 0 1em;
    border-bottom: solid 2px black;
  }

  div.statsHeader {
    border-radius: 5px 5px 0 0;
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

  div.player {
    background-color: #8B2635;
  }

  /* fixtures */

  div.line {
    width: 100%;
    height: 50px;
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    font-size: 1em;
    background: #8B2635;
    border-bottom: 1px solid black;
  }

  div.line:first-of-type {
    border-radius: 5px 5px 0px 0px;
  }

  div.line:last-of-type {
    border: none;
    border-radius: 0px 0px 5px 5px;
  }

  span.team {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    gap: .3em;
  }

  div.line > span {
    display: flex;
    align-items: center;
    gap: .3em;
  }

  div.line > span.homeTeam {
    justify-content: flex-start;
    padding-left: .3em;

  }

  div.line > span.awayTeam {
    justify-content: flex-end;
    padding-right: .3em;
  }

  span.score {
    width: 100%;
    background: #50151e;
    padding: 0 .6em;
    display: grid;
    place-items: center;
  }

  nav .wrapper a::after {
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  height: 4px;
  background: #8B2635;
  content: '';
  opacity: 0;
  -webkit-transition: opacity 0.3s, -webkit-transform 0.3s;
  -moz-transition: opacity 0.3s, -moz-transform 0.3s;
  transition: opacity 0.3s, transform 0.3s;
  -webkit-transform: translateY(10px);
  -moz-transform: translateY(10px);
  transform: translateY(10px);
}

nav a {
  position: relative;
  display: inline-block;
  margin: 15px 15px;
  outline: none;
  color: #fff;
  text-decoration: none;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 100;
  text-shadow: 0 0 1px rgba(255,255,255,0.3);
  font-size: 1.1em;
}

nav a:hover,
nav a:focus {
  outline: none;
}

nav .wrapper a:hover::after,
nav .wrapper a:focus::after {
  opacity: 1;
  -webkit-transform: translateY(0px);
  -moz-transform: translateY(0px);
  transform: translateY(0px);
}

</style>

