<script>
  import '/src/app.css';

  // COMPONENTS
  import Header from './components/Header.svelte';
  import TableField from './components/TableField.svelte';
  import FixturesField from './components/FixturesField.svelte';
  import StatsField from './components/StatsField.svelte';

  // NAVIGATION
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
      statsVisible = false;
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
</script>


<main>
  <Header />
  <nav>
    <div class="wrapper">
      <a href="/" on:click|preventDefault={navHandler}>Fixtures</a>
      <a href="/" on:click|preventDefault={navHandler}>Table</a>
      <a href="/" on:click|preventDefault={navHandler}>Statistics</a>
    </div>
  </nav>
  {#if tableVisible}  
    <TableField />
  {/if}
  {#if fixturesVisible}  
    <FixturesField />
  {/if}
  {#if statsVisible}
    <StatsField />
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

  @media screen and (max-width: 370px) {
    nav a {
      margin: 10px;
      font-size: 1em;
    }
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

