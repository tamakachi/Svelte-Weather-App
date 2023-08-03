<script>

import FancyTempSlider from "./FancyTempSlider.svelte";

import toast, { Toaster } from 'svelte-french-toast';



import { fade } from 'svelte/transition';
import { fly } from 'svelte/transition';
import { quintOut } from 'svelte/easing';
import { blur } from 'svelte/transition';

//remove this import later and from the package.json
import * as animateScroll from "svelte-scrollto";

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
let unitOfWindSpeed = "m/s"
let windDirection = null
// Hides the form when we have the weather data
let hideForm = false

// Populates possibleUserLocations array from the API with multiple locations that match the input
// Then it scrolls down the page
function setPossibleUserLocations(data){
    possibleUserLocations = data
    weHaveLocations = true
    console.log(possibleUserLocations)

    customScrollToBottom()
}

// Function takes degrees and converts them to a cardinal direction then stores it in windDirection
function getCardinalDirection(degrees){
    if(degrees > 11.25 && degrees < 33.75){
        windDirection = "NNE"
    } else if(degrees > 33.75 && degrees < 56.25){
        windDirection = "NE"
    } else if(degrees > 56.25 && degrees < 78.75){
        windDirection = "ENE"
    } else if(degrees > 78.75 && degrees < 101.25){
        windDirection = "E"
    } else if(degrees > 101.25 && degrees < 123.75){
        windDirection = "ESE"
    } else if(degrees > 123.75 && degrees < 146.25){
        windDirection = "SE"
    } else if(degrees > 146.25 && degrees < 168.75){
        windDirection = "SSE"
    } else if(degrees > 168.75 && degrees < 191.25){
        windDirection = "S"
    } else if(degrees > 191.25 && degrees < 213.75){
        windDirection = "SSW"
    } else if(degrees > 213.75 && degrees < 236.25){
        windDirection = "SW"
    } else if(degrees > 236.25 && degrees < 258.75){
        windDirection = "WSW"
    } else if(degrees > 258.75 && degrees < 281.25){
        windDirection = "W"
    } else if(degrees > 281.25 && degrees < 303.75){
        windDirection = "WNW"
    } else if(degrees > 303.75 && degrees < 326.25){
        windDirection = "NW"
    } else if(degrees > 326.25 && degrees < 348.75){
        windDirection = "NNW"
    } else {
        windDirection = "N"
    }
}

// Populates the weatherData object with the weather data from the API

function setWeatherData(data){
    weatherData = data
    getCardinalDirection(weatherData.current.wind_deg)
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
        unitOfWindSpeed = "mph"

    } else {
        unitsOfMeasurement = "metric"
        temperatureUnit = "°C"
        unitOfWindSpeed = "m/s"
    }
}
// Waits 500ms then shows the form and hides the weather data
function redo(){
                        setTimeout(() => {
                          hideForm = !hideForm
                          weHaveLocations = false
                          userInputLocation = null
                        }, 500);
                    }



function customScrollToBottom(){
  //wait 50ms then scroll to the bottom of the page smoothly using vanilla JS
  setTimeout(() => {
    window.scrollTo({
      top: document.body.scrollHeight,
      left: 0,
      behavior: 'smooth'
    });
  }, 50);

}
</script>


<Toaster />
{#if !hideForm}
<div transition:blur={{ amount: 10 }}>
<FancyTempSlider on:message={handleTempComponent} isFahrenheit={isFahrenheit}/>

<br>
<br>
<br>
<img class ="main-logo"src="Stormscout-transformed.png" alt="Stormscout">
</div>
{/if}
<div class="container">
  
  

  {#if !hideForm}
    <form transition:blur={{ amount: 10 }}>
      
        <h2>Enter your location</h2>
        <input class="bigger space" id="location" bind:value={userInputLocation}>
        <br>
        <br>
        <button 
        on:click|preventDefault={()=>{

            let response = fetch('https://api.openweathermap.org/geo/1.0/direct?q=' + userInputLocation +'&limit=5&appid=e7c5f8b9359c36785de51e91035b8fdb');

            response
            .then(data => data.json())
            .then(data => setPossibleUserLocations(data))
            .catch((error) => {
              console.error('Error:', error);
              toast.error('There was an error fetching the data', {
                  position: "bottom-center"
                })
              
             
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
                toast.error('There was an error fetching the data', {
                  position: "bottom-center"
                })
              })
            }}
              >Give me the weather!</button>
        {/if}
    </form>
    {/if}
    {#if weHaveTheWeather}
    <div class="weather container" transition:blur={{ amount: 10 }}>

      <img class="third-size" src="Stormscout-Logo-Only.png" alt="Stormscout">
        <h2 id="final-location">{possibleUserLocations[selectedLocationIndex].name}</h2>
        
        <div class="weather-info">
          <div class="weather-item">
            <p class="bigger">Temperature:</p>
            <img src="temperature2_64.png">
            <p class="weather-data"> {Math.round(weatherData.current.temp)}{temperatureUnit}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Feels like:</p>
            <img src="senses.png">
            <p class="weather-data"> {Math.round(weatherData.current.feels_like)}{temperatureUnit}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Humidity:</p>
            <img src="humidity.png">
            <p class="weather-data"> {weatherData.current.humidity}%</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Wind:</p>
            <img src="wind.png">
            <p class="weather-data"> {weatherData.current.wind_speed} {unitOfWindSpeed}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">Wind direction:</p>
            <img src="vane.png">
            <p class="weather-data"> {windDirection}</p>
          </div>
          <div class="weather-item">
            <p class="bigger">UV Index:</p>
            <img src="sun.png">
            <p class="weather-data"> {weatherData.current.uvi}</p>
          </div>
        </div>
        <button 
        on:click={()=>{
          weHaveTheWeather = false
          redo()
          

          }} 
          class="top-margin">Choose Again</button>
      </div>
  
    {/if}

  </div>

<style scoped>

/* Google Font */
  @import url('https://fonts.googleapis.com/css2?family=Raleway:wght@400;500;700&display=swap');

*{
  font-family: 'Raleway', sans-serif;
}

.third-size{
  width: 30%;
}

.top-margin {
  margin-top: 20px;
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

.space {
  margin:auto;
  padding: auto;
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

/* Responsive elements */

/* Makes the logo smaller on mobile devices */
@media screen and (max-width: 600px) {
  .main-logo{
    width: 75%;
  }
}

</style>