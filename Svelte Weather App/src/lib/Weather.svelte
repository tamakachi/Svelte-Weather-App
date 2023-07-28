<script>

let userInputLocation = null
let possibleUserLocations = null
let weHaveLocations
let selectedLocationIndex
let weHaveTheWeather = false
let selectedLatitude
let seletedLongitude
let weatherData

function setPossibleUserLocations(data){
    possibleUserLocations = data
    weHaveLocations = true
    console.log(possibleUserLocations)
}

function setWeatherData(data){
    weatherData = data
    weHaveTheWeather = true
    console.log(data)
}

</script>

<div class="container">
    <div class="header">
      <h1>Weather App</h1>
    </div>
    <form>
        <h2>Enter your location</h2>
        <input id="location" bind:value={userInputLocation}>
        <br>
        <br>
        <button 
        on:click|preventDefault={()=>{

            let response = fetch('http://api.openweathermap.org/geo/1.0/direct?q=' + userInputLocation +'&limit=5&appid=e7c5f8b9359c36785de51e91035b8fdb');

            response
            .then(data => data.json())
            .then(data => setPossibleUserLocations(data))
            .catch((error) => {
              console.error('Error:', error);
            })}}

        >Go!</button>
            <br>
            <br>
        {#if weHaveLocations}
            <select bind:value={selectedLocationIndex} name="locations">
                {#each possibleUserLocations as posLocation, index}
                <option value={index}>{posLocation.name} - {posLocation.country} - {posLocation.state}</option>
                {/each}
            </select>

            <button 
            on:click|preventDefault={()=>{
             selectedLatitude = possibleUserLocations[selectedLocationIndex].lat
             seletedLongitude = possibleUserLocations[selectedLocationIndex].lon

              let response = fetch('https://api.openweathermap.org/data/2.5/onecall?lat=' + selectedLatitude + '&lon=' + seletedLongitude + '&exclude={part}&appid=e7c5f8b9359c36785de51e91035b8fdb');

              response
              .then(data => data.json())
              .then(data => setWeatherData(data))
              .catch((error) => {
                console.error('Error:', error);
              })
            }}
              >Give me the weather!</button>
        {/if}
        
    </form>
    {#if weHaveTheWeather}
    <div class="weather">
      <p class="temperature">Temperature: 20°C</p>
      <p class="description">Sunny</p>
    </div>
    {/if}
  </div>

<style scoped>

.container {
  width: 400px;
  margin: 0 auto;
}

.header {
  text-align: center;
}

.weather {
  margin-top: 20px;
}

.temperature {
  font-size: 3em;
}

.description {
  font-size: 1.5em;
}

</style>