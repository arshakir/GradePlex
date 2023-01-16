<script>
  import {parseXml} from '../xml2json.js';

  let postal = '';
  let districts = [];

  export let link;

  async function setDistricts(){
    const body = '<?xml version="1.0" encoding="utf-8"?>\
      <soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ProcessWebServiceRequest xmlns="http://edupoint.com/webservices/"><userID>EdupointDistrictInfo</userID><password>Edup01nt</password><skipLoginLog>1</skipLoginLog><parent>0</parent><webServiceHandleName>HDInfoServices</webServiceHandleName><methodName>GetMatchingDistrictList</methodName><paramStr>&lt;Parms&gt;&lt;Key&gt;5E4B7859-B805-474B-A833-FDB15D205D40&lt;/Key&gt;&lt;MatchToDistrictZipCode&gt;' + postal + '&lt;/MatchToDistrictZipCode&gt;&lt;/Parms&gt;</paramStr></ProcessWebServiceRequest></soap:Body></soap:Envelope>';

    const url = 'https://support.edupoint.com/Service/HDInfoCommunication.asmx';
    const response = await fetch(url, {
      method: 'POST', 
      headers: {
        'Content-Type': 'text/xml'
      },
      body: body,
    });

    let results = parseXml(decode(await response.text())).getElementsByTagName("DistrictInfo");

    districts = [];
    for (let i=0; i < results.length; i++){
      districts = [...districts, {name:results[i].attributes.Name.nodeValue, url:results[i].attributes.PvueURL.nodeValue}];
    }
  }

  function decode(input) {
    var txt = document.createElement("textarea");
    txt.innerHTML = input;
    return txt.value;
  }


</script>


<form on:submit|preventDefault={setDistricts}>
  <input bind:value={postal} placeholder="Zip Code">
  <button disabled={postal.length < 3 || postal.length > 5} type=submit> Search </button>

  <select bind:value={link}>
    {#each districts as district}
      <option value={district.url}>
        {district.name}
      </option>
    {/each}
  </select>
</form>
