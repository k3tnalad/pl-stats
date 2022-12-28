<script>
  import { fade, fly } from 'svelte/transition';
  import '/src/app.css';
  
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



$: CURRENT_GAME_WEEK = JSON.parse(localStorage.getItem('cGW')) || null;
$: previousWeek = CURRENT_GAME_WEEK - 1;
$: nextWeek = CURRENT_GAME_WEEK + 1;

const weekHandler = e => {
  let weekNumber = (e.target.textContent).split(' ').pop();
  CURRENT_GAME_WEEK = Number(weekNumber);
  getGameWeek(CURRENT_GAME_WEEK);
}
const lastGWEndpoint = 'https://api-football-v1.p.rapidapi.com/v3/fixtures/rounds?league=39&season=2022&current=true';
const options = {
  method: 'GET',
  headers: {
    'X-RapidAPI-Key': '9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a',
    'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
    }
};

$: fixtures = JSON.parse(localStorage.getItem(`fixtures${CURRENT_GAME_WEEK}`)) || [];


async function getGameWeek(num) {
  const endpoint = `https://api-football-v1.p.rapidapi.com/v3/fixtures?league=39&season=2022&&round=Regular%20Season%20-%20${num}`; 
    const fixturesReq = await fetch(endpoint, options)
    if (fixturesReq.ok) {
        const fixturesData = await fixturesReq.json();
        fixtures = (fixturesData.response).sort((a, b) => a.fixture.timestamp - b.fixture.timestamp);
        localStorage.setItem(`fixtures${CURRENT_GAME_WEEK}`, JSON.stringify(fixtures));
    }
    fixtures = JSON.parse(localStorage.getItem(`fixtures${CURRENT_GAME_WEEK}`));
}

async function lastGameWeek() {
  if (CURRENT_GAME_WEEK) {
    return;
  } else {
    const weekReq = await fetch(lastGWEndpoint, options)
    if (weekReq.ok) {
      const currentWeek = await weekReq.json();
      CURRENT_GAME_WEEK = Number(currentWeek.response[0].split(' ').pop());
      localStorage.setItem('cGW', JSON.stringify(CURRENT_GAME_WEEK))
    }
  }
  getFixturesData();
}
lastGameWeek();

async function getFixturesData() {
  const lastGWfixturesEndpoint = `https://api-football-v1.p.rapidapi.com/v3/fixtures?league=39&season=2022&&round=Regular%20Season%20-%20${CURRENT_GAME_WEEK}`;
  console.log(lastGWfixturesEndpoint);
    if (fixtures.length > 1) {
    return
    } else { 
    const fixturesReq = await fetch(lastGWfixturesEndpoint, options)
    if (fixturesReq.ok) {
        const fixturesData = await fixturesReq.json();
        fixtures = (fixturesData.response).sort((a, b) => a.fixture.timestamp - b.fixture.timestamp);
        localStorage.setItem(`fixtures${CURRENT_GAME_WEEK}`, JSON.stringify(fixtures));
    }
    }

    fixtures = JSON.parse(localStorage.getItem('fixtures'));
} 

</script>

<div class="panel">
  <span class="gameweek">Gameweek {CURRENT_GAME_WEEK}</span>
  <nav>
    <a class="button-59" href="/" on:click|preventDefault={weekHandler}>Previous week {previousWeek}</a>
    <a class="button-59" href="/" on:click|preventDefault={weekHandler}>Next week {nextWeek}</a>
  </nav>
</div>

<div in:fly="{{duration: 200}}" out:fade="{{ duration: 100}}" class="fixtures">
    {#each fixtures as fixture}
    <div class="line" id={fixture.fixture.id}>
        <span class="homeTeam">
          <img src="{fixture.teams.home.logo}" alt="teamLogo">
          <span class="team-name">{nameShortener(fixture.teams.home.name)}</span>
        </span>
        <span class="score">
          { fixture.goals.home == null ?
            `T B P` :
            `${fixture.goals.home} - ${fixture.goals.away}`
          }
        </span>
        <span class="awayTeam">
          <span class="team-name">{nameShortener(fixture.teams.away.name)}</span>
          <img src="{fixture.teams.away.logo}" alt="teamLogo">
        </span>
      </div>
    {/each}
</div>

<style>
    div.fixtures {
    width: 95%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    color: #040F16;
    box-shadow: rgba(0, 0, 0, 0.25) 0px 14px 28px, rgba(0, 0, 0, 0.22) 0px 10px 10px;
    border-radius: 5px 5px 5px 5px;
  }

  div.panel {
    width: 95%;
    background-color: #72727E;
    border-radius: 3px;
    padding: 1em;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    gap: 1em;
  }

  @media screen and (min-width: 640px) {
    div.fixtures, div.panel {
      width: 60%;
    }
   }

  div.line {
    width: 100%;
    height: 50px;
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    font-size: 1em;
    background: #72727E;
    border-bottom: 2px solid black;
  }

  div.line:first-of-type {
    border-radius: 5px 5px 0px 0px;
  }

  div.line:last-of-type {
    border: none;
    border-radius: 0px 0px 5px 5px;
  }

  div.line > span {
    display: flex;
    align-items: center;
    gap: .3em;
  }

  div.line > span.homeTeam {
    justify-content: flex-start;
    padding-left: .5em;

  }

  div.line > span.awayTeam {
    justify-content: flex-end;
    padding-right: .5em;
  }

  span.score {
    width: 100%;
    background: #4E4E56;
    padding: 0 .6em;
    display: grid;
    place-items: center;
  }

  span.gameweek {
    font-size: large;
  }

  div.panel > nav {
    width: 100%;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
  }

 .button-59 {
  align-items: center;
  background-color: #FBFBFF;
  border: 2px solid #000;
  color: #040F16;
  cursor: pointer;
  display: inline-flex;
  fill: #000;
  font-size: 16px;
  height: 48px;
  justify-content: center;
  letter-spacing: -.8px;
  line-height: 24px;
  min-width: 140px;
  outline: 0;
  padding: .4em;
  text-align: center;
  text-decoration: none;
  transition: all .3s;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
}

.button-59:hover {
  box-shadow: rgba(0, 0, 0, 0.25) 0px 14px 28px, rgba(0, 0, 0, 0.22) 0px 10px 10px;
  background-color: #72727E;
}

</style>

