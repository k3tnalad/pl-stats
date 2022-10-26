<script>

  const endpoint =  "https://api-football-v1.p.rapidapi.com/v3/standings?season=2022&league=39";
  const headers = {
		    "x-rapidapi-host": "api-football-v1.p.rapidapi.com",
		    "x-rapidapi-key": "9b629c4000msh5f1e9f22f14de23p12cb4cjsn5860dd1d131a"
  }

  let standings = JSON.parse(localStorage.getItem('teams')) || []; 
  console.log(standings)

  async function getLeagueData() {
    if (standings.length > 1) {
      return
    } else { 
      const response = await fetch(endpoint, {
        'method': "GET",
        headers,
      })
      const data = await response.json();
      console.log('fetched')
      let arrayData = data.response[0].league.standings[0];
      standings = arrayData;
      localStorage.setItem('teams', JSON.stringify(arrayData));
      console.log('stored');
    }
    standings = JSON.parse(localStorage.getItem('teams'))
  } 

  getLeagueData();

</script>

<main>
  <div class="table">
    <div class="header">
      <span>Pos</span>
      <span>Team</span>
      <span>+/-</span>
      <span>Points</span>
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
</main>

<style>
  div.table {
    width: 650px;
    border: 2px white solid;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 2px;
  }

  div.position, div.header {
    width: 100%;
    color: #F2F4F3;
    display: grid;
    grid-template-columns: 10% 50% 30% 10%;
    background-color: #22333B;
  }
</style>
