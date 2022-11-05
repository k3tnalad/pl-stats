<script>
    import { fade, fly } from 'svelte/transition';
    import '/src/app.css';

    const fixturesEndpoint = "https://api-football-v1.p.rapidapi.com/v3/fixtures?league=39&season=2022&last=10";
    const options = {
        method: 'GET',
        headers: {
        'X-RapidAPI-Key': '9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a',
        'X-RapidAPI-Host': 'api-football-v1.p.rapidapi.com'
        }
    };


    let fixtures = JSON.parse(localStorage.getItem('fixtures')) || [];
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


    async function getFixturesData() {
    if (fixtures.length > 1) {
      return
    } else { 
      const fixturesReq = await fetch(fixturesEndpoint, options)
      if (fixturesReq.ok) {
        const fixturesData = await fixturesReq.json();
        fixtures = fixturesData.response;
        localStorage.setItem('fixtures', JSON.stringify(fixtures));
      }
    }

    fixtures = JSON.parse(localStorage.getItem('fixtures'))
  } 

  getFixturesData();
</script>


<div in:fly="{{duration: 200}}" out:fade="{{ duration: 100}}" class="fixtures">
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

<style>
    div.fixtures {
    width: 95%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
  }

  @media screen and (min-width: 640px) {
    div.fixtures {
      width: 60%;
    }
   }

  div.line {
    width: 100%;
    height: 50px;
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    font-size: 1em;
    background: #8B2635;
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
</style>

