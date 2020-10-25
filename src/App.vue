<template>
    <div id="app">
            <div class="weatherNow" v-if="typeof localWeather.current !== 'undefined'">
                <h3>{{current}}</h3>
                <p>{{ Math.round(localWeather.current.temp - 273.15)}}°</p>
            </div>
            
            <button class="locateBtn" @click="locateMe">Get Location</button>

            <div v-if="errorStr">
                Sorry, but the following error
                occurred: {{errorStr}}
            </div>
            
            <div v-if="gettingLocation">
                <i>Getting your location...</i>
            </div>

            <input 
                type="text" 
                class="search" 
                v-model="query" 
                placeholder="Lisbon,pt"
                @keypress="getWeather"
            />

            <div class="weatherNow" v-if="typeof weather.main !== 'undefined'">
                <h3>{{ weather.name }}, {{ weather.sys.country }}</h3>
                <p>{{ Math.round(weather.main.temp - 273.15)}}°</p>
            </div>
    </div>
</template>

<script>
export default {
    name: 'App',
    data () {
        return {
            apiKey: '&appid=d999c28416a3ddfa9333ba6b67985a20',
            url: 'https://api.openweathermap.org/data/2.5/',
            query: '',
            weather: {},
            localWeather: {},
            location:null,
            gettingLocation: false,
            errorStr: null,
            lat: '38.7223',
            lon: '9.1393',
            current: "Lisbon, PT"
        }
    },
    methods: {
        getLocalWeather () {
            fetch(`${this.url}onecall?lat=${this.lat}&lon=${this.lon}${this.apiKey}`)
                .then( res => {
                    return res.json()
                })
                .then(this.showLocalWeather);
        },

        showLocalWeather (res) {
            this.localWeather = res;
        },

        getWeather (e) {
            if (e.key == "Enter"){
                fetch(`${this.url}weather?q=${this.query}${this.apiKey}`)
                    .then( res => {
                        return res.json()
                    })
                    .then(this.showWeather);
            }
        },

        showWeather (res) {
            this.weather = res;
        },

        async getLocation() {
            return new Promise((resolve, reject) => {
                navigator.geolocation.getCurrentPosition(pos => {
                    this.lat = pos.coords.latitude;
                    this.lon = pos.coords.longitude;
                    this.current = "Your Location";
                    this.getLocalWeather()
                    resolve(pos);
                }, err => {
                    reject(err);
                });
            });
        },

        async locateMe() {
            this.gettingLocation = true;
            try {
                this.gettingLocation = false;
                this.location = await this.getLocation();
            } catch(e) {
                this.gettingLocation = false;
                this.errorStr = e.message;
            }
        }
    },

    mounted(){
        this.getLocalWeather()
    },
}
</script>

<style>
#app {
    display: flex;
    justify-content: center;
    flex-direction: column;
}

.locateBtn {
    margin: 20px auto;
    width: 80px;
}

div {
    margin: 0 auto;
}

.search {
    margin: 0 auto;
    width: 30%;
    text-align: center;
}

.weatherNow h3, p{
    text-align: center;
}
</style>
