<script>
  export let student;
  function handleSubmit() {
    student.error = false;
    student.loggedin = true;
    if (student.remember){
      localStorage.setItem("student", JSON.stringify(student));
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit}>
  <input name="userID" bind:value={student.userid} placeholder="UserID">
  <input type="password" name="password" bind:value={student.passwd} placeholder="Password">
  <button disabled={!student.userid || !student.passwd || !student.url} type=submit> Log In </button>
  <label>
    <input type=checkbox bind:checked={student.remember}>
      Remember Me
  </label>
</form>
