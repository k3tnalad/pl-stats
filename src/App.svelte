<script>
  import '/src/app.css';

  // COMPONENTS
  import Header from './components/Header.svelte';
  import TableField from './components/TableField.svelte';
  import FixturesField from './components/FixturesField.svelte';
  import StatsField from './components/StatsField.svelte';
  import Footer from './components/Footer.svelte';

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
  <div class="data">
    <nav>
      <div class="wrapper">
        <button on:click|preventDefault={navHandler}>Fixtures</button>
        <button on:click|preventDefault={navHandler}>Table</button>
        <button on:click|preventDefault={navHandler}>Statistics</button>
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
  </div>
  <Footer />
</main>

<style>
  main {
    width: 100%;
    height: 100%;
    display: grid;
    grid-template-rows: 1fr auto 1fr;
  }
  main div.data {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: center;
    flex-direction: column;
    gap: 1rem;  
    padding: 1em 0 2em 0;
  }

  button {
    padding: .5em 1em;
    border: 2px solid black;
    background: none;
    font-size: 1.1em;
  }

  button:hover, button:focus {
    animation: lightFill 200ms ease-in forwards;
    cursor: pointer;
  }

  @keyframes lightFill {
    from {
      background: none;
      color: #040F16;
    }
    to {
      background: #59656F;
      color: #FBFBFF;
    }
  }
  </style>