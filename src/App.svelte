<script>
  let student = JSON.parse(localStorage.getItem('student')) || {};
  import District from './components/District.svelte';
  import LogIn from './components/LogIn.svelte';
  import Gradebook from './components/Gradebook.svelte';
  import Icon from '@iconify/svelte';

</script>

{#if !student.loggedin}
  <h1> GradePlex </h1>
  <h3> Log In </h3>
  {#if student.error}
    <p> Error logging in try again </p>
  {/if}
  <District bind:link={student.url}/><br>
  <LogIn bind:student={student}/>
  <a href="https://github.com/arshakir"> <Icon icon="mdi:github" inline=true/> Github </a>
  <h3> How to add GradePlex to the Home Screen </h3>
  <ol>
    <li> Open your browser’s settings. </li>
    <li> Scroll down and tap Add to Home screen. </li>
  </ol>
{:else}
  <Gradebook bind:student={student}/>  
{/if}
