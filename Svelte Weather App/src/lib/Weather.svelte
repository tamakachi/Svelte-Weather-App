<script>
import FancyTempSlider from "./FancyTempSlider.svelte";
import { fade } from 'svelte/transition';
import { fly } from 'svelte/transition';


let userInputLocation = null
let possibleUserLocations = null


let weHaveLocations
let selectedLocationIndex
let weHaveTheWeather = false

// For the geolocation API
let selectedLatitude
let selectedLongitude

// Holds the weather data after it comes back from the API
let weatherData

// Metric and imperial variables
let isFahrenheit = false
let unitsOfMeasurement = "metric"
let temperatureUnit = "°C"

// Hides the form when we have the weather data
let hideForm = false


// Populates possibleUserLocations array from the API with multiple locations that match the input
function setPossibleUserLocations(data){
    possibleUserLocations = data
    weHaveLocations = true
    console.log(possibleUserLocations)
}

function setWeatherData(data){
    weatherData = data
    console.log(weatherData)
    // wait 550 milliseconds so the transition can finish before showing the data
    setTimeout(() => {
        weHaveTheWeather = true
    }, 550);
    
    hideForm = true
    console.log(data)
}

// Changes the units of measurement when the FancyTempSlider component dispatches an event caused by the user clicking the slider in the component

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
{#if !hideForm}
<div transition:fade={{ duration: 500 }}>
<FancyTempSlider on:message={handleTempComponent}/>

<br>
<br>
<br>
<img src="..\src\assets\Stormscout-transformed.png" alt="Stormscout">
</div>
{/if}
<div class="container">
  

  {#if !hideForm}
    <form transition:fade={{ duration: 500 }}>
      
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
    {/if}
    {#if weHaveTheWeather}
    <div class="weather container" transition:fly={{ y: 200, duration: 1000 }}>
      
        <h2 id="final-location">{possibleUserLocations[selectedLocationIndex].name}</h2>
        
        <div class="weather-info">
          <div class="weather-item">
            <p class="bigger">Temperature:</p>
            <img src="..\public\temperature2_64.png">
            <p class="weather-data"> {Math.round(weatherData.current.temp)}{temperatureUnit}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Feels like:</p>
            <img src="..\public\senses.png">
            <p class="weather-data"> {Math.round(weatherData.current.feels_like)}{temperatureUnit}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Humidity:</p>
            <img src="..\public\humidity.png">
            <p class="weather-data"> {weatherData.current.humidity}%</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Wind:</p>
            <img src="..\public\wind.png">
            <p class="weather-data"> {weatherData.current.wind_speed}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Wind direction:</p>
            <img src="..\public\vane.png">
            <p class="weather-data"> {weatherData.current.wind_deg}°</p>
          </div>
          <div class="weather-item">
            <p class="bigger">UV Index:</p>
            <img src="..\public\sun.png">
            <p class="weather-data"> {weatherData.current.uvi}</p>
          </div>
        </div>
      </div>
  
    {/if}

    

<!--  Dummy data for testing purposes
  
  <div class="weather container" >
  <h2>Richards Bay</h2>
  
  <div class="weather-info">
    <div class="weather-item">
      <p class="bigger">Temperature:</p>
      <img src="..\public\temperature2_64.png">
      <p class="weather-data"> 24°C</p>
    </div>
    <div class="weather-item">
      <p class="bigger">Feels like:</p>
      <img src="..\public\senses.png">
      <p class="weather-data"> 56°C</p>
    </div>
    <div class="weather-item">
      <p class="bigger">Humidity:</p>
      <img src="..\public\humidity.png">
      <p class="weather-data"> 50%</p>
    </div>
    <div class="weather-item">
      <p class="bigger">Wind:</p>
      <img src="..\public\wind.png">
      <p class="weather-data"> 8.57</p>
    </div>
    <div class="weather-item">
      <p class="bigger">Wind direction:</p>
      <img src="..\public\vane.png">
      <p class="weather-data"> 11°</p>
    </div>
    <div class="weather-item">
      <p class="bigger">UV Index:</p>
      <img src="..\public\uv.png">
      <p class="weather-data"> 0</p>
    </div>
  </div>
</div> -->

  </div>

<style scoped>

/* Google Font */
  @import url('https://fonts.googleapis.com/css2?family=Raleway:wght@400;500;700&display=swap');

*{
  font-family: 'Raleway', sans-serif;
}

#final-location{
  
  font-weight: 400;
  font-size: 2.5em;

}
.weather-data {
  margin: 0;
  font-size: 1.4em;
  font-weight: 500;
  color:#333;
  margin-left: 25% !important;
  margin-right: 25% !important;
  
}

.bigger {
  font-size: 1.25em;
  font-weight: 500;
  color:#444;
}

.weather-info {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
 .weather-item {
  flex-basis: calc(50% - 10px);
  margin-bottom: 10px;
  box-sizing: border-box;
}


.container {
  width: 400px;
  margin: 0 auto;
}

h2{
  font-weight: 400;
  font-size: 1.75em;
}



.weather {
  margin-top: 20px;
  size: 2em;

}

.weather p {
  text-align: center;
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