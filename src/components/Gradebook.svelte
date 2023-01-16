<script>
  import {parseXml} from '../xml2json.js';
  import { onMount } from 'svelte';
  import Sidebar from './Sidebar.svelte';
  import Icon from '@iconify/svelte';
  export let student;
  student.period = -1;

  let selected = -1;
  let classes = [];
  let periods = [];

  async function getGrades(){
    periods = [];
    classes = [];

    var body = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ProcessWebServiceRequest xmlns="http://edupoint.com/webservices/"><userID>' + student.userid + '</userID><password>' + student.passwd + '</password><skipLoginLog>1</skipLoginLog><parent>0</parent><webServiceHandleName>PXPWebServices</webServiceHandleName><methodName>Gradebook</methodName><paramStr>&lt;Parms&gt;&lt;ChildIntID&gt;0&lt;/ChildIntID&gt;&lt;/Parms&gt;</paramStr></ProcessWebServiceRequest></soap:Body></soap:Envelope>';

    if (student.period != -1){
      body = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ProcessWebServiceRequest xmlns="http://edupoint.com/webservices/"><userID>' + student.userid + '</userID><password>' + student.passwd + '</password><skipLoginLog>1</skipLoginLog><parent>0</parent><webServiceHandleName>PXPWebServices</webServiceHandleName><methodName>Gradebook</methodName><paramStr>&lt;Parms&gt;&lt;ChildIntID&gt;0&lt;/ChildIntID&gt;&lt;ReportPeriod&gt;' + student.period  + '&lt;/ReportPeriod&gt;&lt;/Parms&gt;</paramStr></ProcessWebServiceRequest></soap:Body></soap:Envelope>';
    } 

    const response = await fetch(student.url + '/Service/PXPCommunication.asmx', {
      method: 'POST', 
      headers: {
        'Content-Type': 'text/xml'
      },
      body: body,
    });

    let y = await response.text();

    if (y.match("ERROR")){
      student.error = true;
      student.loggedin = false;
    } 

    let x = parseXml(decode(y));
    let p = x.getElementsByTagName("ReportPeriod");
    for (let i=0; i < p.length; i++){
      periods = [...periods, p[i].attributes.GradePeriod.nodeValue]
    }
    student.period = student.period == -1 ? periods.length-1 : student.period;
    console.log(periods);

    let c = x.getElementsByTagName("Course");
    for (let i=0; i < c.length; i++){
      let assignments = [];
      let a = c[i].getElementsByTagName("Assignment");

      for (let j=0; j < a.length; j++){
        assignments.push({name:a[j].attributes.Measure.nodeValue, type:a[j].attributes.Type.nodeValue});

        let ind = a[j].attributes.Points.nodeValue.search("/");

        if (ind != -1){
          assignments[j].score = parseFloat(a[j].attributes.Points.nodeValue.substring(0,ind));
          assignments[j].total = parseFloat(a[j].attributes.Points.nodeValue.substring(ind+1));
        } else {
          assignments[j].total = parseFloat(a[j].attributes.Points.nodeValue);
          assignments[j].score = null;
        }
      }

      classes = [...classes, {title:c[i].attributes.Title.nodeValue.substring(0,c[i].attributes.Title.nodeValue.indexOf("(")), period:c[i].attributes.Period.nodeValue, teacher:c[i].attributes.Staff.nodeValue, letter:c[i].children[0].children[0].attributes.CalculatedScoreString.nodeValue, percent:c[i].children[0].children[0].attributes.CalculatedScoreRaw.nodeValue, assignments:assignments}];

    }
  }

  function decode(input) {
    var txt = document.createElement("textarea");
    txt.innerHTML = input;
    return txt.value.replace("amp;","");
  }


  onMount(async () => {
    getGrades();
  });

  $: updatePercent(classes);

  function updatePercent(classes){
    for (let c in classes){
      classes[c].allTask = 0;
      classes[c].totalAllTask = 0;
      classes[c].practice = 0;
      classes[c].totalPractice = 0;
      for (let a in classes[c].assignments){
        if (classes[c].assignments[a].score !== null){
          if (classes[c].assignments[a].type === 'All Tasks / Assessments'){
            classes[c].allTask += classes[c].assignments[a].score;
            classes[c].totalAllTask += classes[c].assignments[a].total;
          } else if (classes[c].assignments[a].type === 'Practice / Preparation'){
            classes[c].practice += classes[c].assignments[a].score;
            classes[c].totalPractice += classes[c].assignments[a].total;
          }
        }
      }

      if (classes[c].totalPractice == 0){
        classes[c].percent = (classes[c].allTask/classes[c].totalAllTask * 100).toFixed(2);
      } else if (classes[c].totalAllTask == 0){
        classes[c].percent = (classes[c].practice/classes[c].totalPractice* 100).toFixed(2);
      }else {
        classes[c].percent = (((classes[c].allTask/classes[c].totalAllTask) * 0.9 + (classes[c].practice/classes[c].totalPractice) * 0.1)*100).toFixed(2);
      }
    }
  }

  $: updateLetter(classes);

  function updateLetter(classes){
    for (let c in classes){
      let ascii = 69 - Math.floor((classes[c].percent - 49.5)/10);

      if (ascii < 65){
        ascii = 65;
      } else if (ascii > 69){
        ascii = 69
      }

      classes[c].letter = String.fromCharCode(ascii);
    }
  }
  $: console.log(student.period);

</script>

<Sidebar {classes} bind:selected={selected}/>

<div style="float:right">
<select bind:value={student.period} on:change={() => {getGrades()}}>
  {#each periods as period, i}
    <option value={i}>
      {period}
    </option>
  {/each}
</select>
<button on:click={getGrades}> <Icon icon="material-symbols:refresh" inline=true /> Refresh </button>
<button on:click={() => {student = {}; localStorage.clear()}}> <Icon icon="material-symbols:logout" inline=true /> Log Out </button>
</div>

{#if classes.length == 0}
<br> <p class="load"> Loading... </p>
{:else}
{#if selected==-1 }
<table>
      <th> <h3>Class </h3> </th>
      <th> <h3> Period </h3> </th>
      <th> <h3> Grade </h3> </th>
      {#each classes as c,i}
        <tr>
          <td> {c.title} </td>
          <td> {c.period} </td>
          <td> {c.letter} </td>
          <td> {c.percent}% </td>
          </tr>
          {/each}
</table>
  {:else}
<h1> {classes[selected].title} </h1>
    <h3> {classes[selected].teacher} </h3>
    <h1> {classes[selected].letter} </h1>
    <h1> {classes[selected].percent}% </h1>

<button on:click={() => {classes[selected].assignments = [{name: 'New Assignment', type: 'All Tasks / Assessments', score: 10, total: 10}, ...classes[selected].assignments]}}> Add New All Tasks Assignment </button>
<button on:click={() => {classes[selected].assignments = [{name: 'New Assignment', type: 'Practice / Preparation', score: 10, total: 10}, ...classes[selected].assignments]}}> Add New Practice Preparation Assignment </button>

<table>
  <th>Assignment</th>
  <th>Type</th>
  <th>Score</th>
  {#each classes[selected].assignments as a,i}
    <tr>
      <td> {a.name} </td>
      <td> {a.type} </td>
      <td><input type=number bind:value={a.score} size="4">/<input type=number bind:value={a.total} size="4"></td>
      <td> {(a.score / a.total * 100).toFixed(0)}% </td>
      <td> <button on:click={() => {classes[selected].assignments = [...classes[selected].assignments.slice(0,i), ...classes[selected].assignments.slice(i+1)] }}> Remove </button></td>
    </tr>
  {/each}
</table>

{/if}
{/if}

<style> 
  p{
    left: 0;
    position:absolute;
  }
</style>
