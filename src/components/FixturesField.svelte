<script>
  import { fade, fly } from 'svelte/transition';
  import '/src/app.css';
  
const nameShortener = (string) => {
    if (string === 'Nottingham Forest') {
      return 'N. Forest'
      } else if (string === 'Manchester City') {
      return 'Man City'
      } else if (string === 'Manchester United') {
        return 'Man United'
      } else if (string === 'Crystal Palace') {
        return 'C. Palace';
      } else {
        return string;
      }
}
const getTime = (date) => {
  let [day, time] = date.split('T');
  let properTime = time.split('').splice(0, 5).join('');
  return `${properTime}`;
}

let clicked = true;

$: CURRENT_GAME_WEEK = JSON.parse(localStorage.getItem('cGW')) || null;
$: previousWeek = CURRENT_GAME_WEEK - 1;
$: nextWeek = CURRENT_GAME_WEEK + 1;
let fixtures = JSON.parse(localStorage.getItem(`fixtures${CURRENT_GAME_WEEK}`)) || [];

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

    fixtures = JSON.parse(localStorage.getItem(`fixtures${CURRENT_GAME_WEEK}`));
} 

</script>

<div class="panel" in:fly="{{x: -200, duration: 400}}" out:fade="{{ duration: 100}}">
  <nav>
    <button on:click|preventDefault={weekHandler}>Prev {previousWeek}</button>
    <span class="gameweek">GW {CURRENT_GAME_WEEK}</span>
    <button on:click|preventDefault={weekHandler}>Next {nextWeek}</button>
  </nav>
</div>
 
<div in:fly="{{x: -200, duration: 500}}" out:fade="{{ duration: 100}}" class="fixtures">
    {#each fixtures as fixture}
    <div class="line" id={fixture.fixture.id}>
        <span class="homeTeam">
          <img src="{fixture.teams.home.logo}" alt="teamLogo">
          <span class="team-name">{nameShortener(fixture.teams.home.name)}</span>
        </span>
        <span class="score">
          {#if fixture.score.fulltime.home === null}
            {getTime(fixture.fixture.date)}
          {:else}
            {fixture.score.fulltime.home} - {fixture.score.fulltime.away}
          {/if}
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
    background-color: #EDF2F4;
    color: #040F16;
    box-shadow: rgba(0, 0, 0, 0.25) 0px 14px 28px, rgba(0, 0, 0, 0.22) 0px 10px 10px;
    border-radius: 5px 5px 5px 5px;
  }

  div.panel {
    width: 95%;
    border-radius: 3px;
    padding: .5em;
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
    background: #C4D4CA;
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
    background: #59656F;
    padding: 0 .6em;
    display: grid;
    place-items: center;
  }
  span.gameweek {
    font-size: 18px;
  }

  div.panel > nav {
    width: 100%;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
  }
  button {
    width: 120px;
    height: 50px;
    border: 2px solid black;
    font-size: 1em;
    background: #FBFBFF;
  }

  button:hover {
    animation: lightFill 200ms ease-in forwards;
    cursor: pointer;
  }

  @keyframes lightFill {
    from {
      color: #040F16;
      background: #FBFBFF;
    }
    to {
      background: #59656F;
      color: #FBFBFF;
    }
  }

</style>

