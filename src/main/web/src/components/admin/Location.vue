<template>
    <div class="weather">
        <h3>Weather</h3>
        <b-card>
            <form @submit.prevent="saveLocation">
                <div class="form-group">
                    <label for="forecastUrl">Link to the page <a target="_blank" href="http://www.openweathermap.org">Openweathermap.org</a> from the city</label>
                    <input type="text" text="http://api.openweathermap.org/data/2.5/weather?q=Bangalore,IN&APPID=95fa3a339bc9eeec51d0daf6d5ece591"
                    	:disabled=true
                    	v-model="location.weatherForecastUrl" class="form-control" id="forecastUrl">
                </div>

                <b-alert :show="saveError != null" variant="danger">
                    {{saveError}}
                </b-alert>

                <div class="centered-btn-wrapper">
                    <b-button type="submit" class="save-btn"
                              :disabled="!location.weatherForecastUrl"
                              variant="primary">
                        Record
                    </b-button>
                </div>
            </form>
        </b-card>
    </div>
</template>

<script>
import {locationService} from '@/services/LocationService.js'

export default {
    data() {
        return {
            saveError: null,
            location: {weatherForecastUrl: null}
        }
    },

    mounted() {
        locationService.getLocation().then((result) => {
            if (result) {
                this.location.weatherForecastUrl = result.weatherForecastUrl;
            }
        });
    },

    methods: {
        saveLocation() {
            this.saveError = null;
            locationService.setLocation(this.location).then((result) => {
                if (!result) {
                    this.saveError = 'Backup failed';
                }
            }, (result) => {
                this.saveError = 'Backup failed';
            });
        }
    }
}

</script>

<style scoped>
.weather {
    margin-top: 10px;
    margin-bottom: 10px;
}
</style>
