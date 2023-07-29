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
<div class="container">
    



      <h1>Weather</h1>
    
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
@import url('https://fonts.googleapis.com/css2?family=Raleway&display=swap');

*{
  font-family: 'Raleway', sans-serif;
}
.container {
  width: 400px;
  margin: 0 auto;
}

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
  font-weight: 600;
  letter-spacing: 2px;
  background: transparent;
  color: #fff;
  overflow: hidden;
  box-shadow: 0 0 0 0 transparent;
  text-align: center;
}

/* Fancy button CSS */

button {
  position: relative;
  padding: 10px 20px;
  border-radius: 7px;
  border: 1px solid rgb(61, 106, 255);
  font-size: 14px;
  text-transform: uppercase;
  font-weight: 600;
  letter-spacing: 2px;
  background: transparent;
  color: #fff;
  overflow: hidden;
  box-shadow: 0 0 0 0 transparent;
  -webkit-transition: all 0.2s ease-in;
  -moz-transition: all 0.2s ease-in;
  transition: all 0.2s ease-in;
}

button:hover {
  background: rgb(61, 106, 255);
  box-shadow: 0 0 30px 5px rgba(0, 142, 236, 0.815);
  -webkit-transition: all 0.2s ease-out;
  -moz-transition: all 0.2s ease-out;
  transition: all 0.2s ease-out;
}

button:hover::before {
  -webkit-animation: sh02 0.5s 0s linear;
  -moz-animation: sh02 0.5s 0s linear;
  animation: sh02 0.5s 0s linear;
}

button::before {
  content: '';
  display: block;
  width: 0px;
  height: 86%;
  position: absolute;
  top: 7%;
  left: 0%;
  opacity: 0;
  background: #fff;
  box-shadow: 0 0 50px 30px #fff;
  -webkit-transform: skewX(-20deg);
  -moz-transform: skewX(-20deg);
  -ms-transform: skewX(-20deg);
  -o-transform: skewX(-20deg);
  transform: skewX(-20deg);
}

@keyframes sh02 {
  from {
    opacity: 0;
    left: 0%;
  }

  50% {
    opacity: 1;
  }

  to {
    opacity: 0;
    left: 100%;
  }
}

button:active {
  box-shadow: 0 0 0 0 transparent;
  -webkit-transition: box-shadow 0.2s ease-in;
  -moz-transition: box-shadow 0.2s ease-in;
  transition: box-shadow 0.2s ease-in;
}


select {
  width: 100%;
  padding: 10px;
  border-radius: 7px;
  border: 1px solid rgb(61, 106, 255);
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 2px;
  background: transparent;
  color: #fff;
  overflow: hidden;
  box-shadow: 0 0 0 0 transparent;
  -webkit-transition: all 0.2s ease-in;
  -moz-transition: all 0.2s ease-in;
  transition: all 0.2s ease-in;
}


</style>