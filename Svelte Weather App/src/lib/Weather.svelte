<script>
import FancyTempSlider from "./FancyTempSlider.svelte";


let userInputLocation = null
let possibleUserLocations = null
let weHaveLocations
let selectedLocationIndex
let weHaveTheWeather = false
let selectedLatitude
let selectedLongitude
let weatherData
let isFahrenheit = false
let unitsOfMeasurement = "metric"
let temperatureUnit = "°C"

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

function handleTempComponent(event){
    isFahrenheit = event.detail.text
    if(isFahrenheit){
        unitsOfMeasurement = "imperial"
        temperatureUnit = "°F"
    } else {
        unitsOfMeasurement = "metric"
        temperatureUnit = "°C"
    }


}

</script>

<FancyTempSlider on:message={handleTempComponent}/>
<br>
<br>
<br>
<img src="..\src\assets\Stormscout-transformed.png">
<div class="container">
    
    <form>
        <h2>Enter your location</h2>
        <input class="bigger" id="location" bind:value={userInputLocation}>
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
            <select class='bigger' 
            bind:value={selectedLocationIndex} name="locations">
                {#each possibleUserLocations as posLocation, index}
                <option value={index}>{posLocation.name} - {posLocation.country} - {posLocation.state}</option>
                {/each}
            </select>
            <br>
            <br>
            <button 
            on:click|preventDefault={()=>{
             selectedLatitude = possibleUserLocations[selectedLocationIndex].lat
             selectedLongitude = possibleUserLocations[selectedLocationIndex].lon

              let response = fetch('https://api.openweathermap.org/data/2.5/onecall?lat=' + selectedLatitude + '&lon=' + selectedLongitude + '&units='+ unitsOfMeasurement + '&exclude={part}&appid=e7c5f8b9359c36785de51e91035b8fdb');

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
      <p class="bigger">Temperature: {Math.round(weatherData.current.temp)}{temperatureUnit}</p>
      <p class="bigger">Feels like: {Math.round(weatherData.current.feels_like)}{temperatureUnit}</p>
      <p class="bigger">Humidity: {weatherData.current.humidity}%</p>
      <p class="bigger">Wind: {weatherData.current.wind_speed}</p>
      <p class="bigger">Wind direction: {weatherData.current.wind_deg}°</p>
      <p class="bigger">UV Index Right Now: {weatherData.current.uvi}</p>
    </div>
    {/if}
  </div>

<style scoped>

/* Google Font */
  @import url('https://fonts.googleapis.com/css2?family=Raleway:wght@400;500;700&display=swap');

*{
  font-family: 'Raleway', sans-serif;
}
.container {
  width: 400px;
  margin: 0 auto;
}

h2{font-weight: 400;}

.header {
  text-align: center;
}

.weather {
  margin-top: 20px;
  size: 2em;
}

.bigger{
  font-size: 1.5em;
}

.temperature {
  font-size: 3em;
}

.description {
  font-size: 1.5em;
}

input {
  width: 100%;
  padding: 10px;
  border-radius: 7px;
  border: 1px solid rgb(61, 106, 255);
  font-size: 14px;
  font-weight: 500;
  letter-spacing: 2px;
  background: transparent;
  color: black;
  overflow: hidden;
  box-shadow: 0 0 0 0 transparent;
  text-align: center;
}

/* Fancy button CSS */
button {
  padding: 1em 2em;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  letter-spacing: 5px;
  text-transform: uppercase;
  color: #2596be;
  transition: all 1000ms;
  font-size: 15px;
  position: relative;
  overflow: hidden;
  outline: 2px solid #2596be;
}

button:hover {
  color: #ffffff;
  transform: scale(1.1);
  outline: 2px solid #70bdca;
  box-shadow: 4px 5px 17px -4px #268391;
}

button::before {
  content: "";
  position: absolute;
  left: -50px;
  top: 0;
  width: 0;
  height: 100%;
  background-color: #2596be;
  transform: skewX(45deg);
  z-index: -1;
  transition: width 1000ms;
}

button:hover::before {
  width: 250%;
}


select {
  width: 100%;
  padding: 10px;
  border-radius: 7px;
  border: 1px solid rgb(61, 106, 255);
  font-size: 14px;
  font-weight: 500;
  letter-spacing: 2px;
  background: transparent;
  color: black;
  overflow: hidden;
  box-shadow: 0 0 0 0 transparent;
  -webkit-transition: all 0.2s ease-in;
  -moz-transition: all 0.2s ease-in;
  transition: all 0.2s ease-in;
}


</style>