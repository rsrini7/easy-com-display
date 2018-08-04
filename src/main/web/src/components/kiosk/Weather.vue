<template>
    <div class="weather">
        <div v-if="weatherForecast">
            <div class="temperature">
                <span class="temperature"> Temp: {{weatherForecast.temperature}}c</span>
            </div>
            <div class="sky-and-rain">
                <div class="sky">Humidity: {{weatherForecast.humidity}}</div>
            </div>
        </div>
    </div>
</template>

<script>
import { cmsService } from '@/services/CmsService.js';

export default {
    data() {
        return {
            weatherForecast: null
        };
    },

    mounted() {
        this.refresh();
        this.refreshId = window.setInterval(() => this.refresh(), 500000);
    },

    beforeDestroy() {
        if (this.refreshId) {
            window.clearInterval(this.refreshId);
        }
    },

    computed: {
        translatedPeriod() {
            if (this.weatherForecast.period === 'morning') {
                return 'Morning';
            } else if (this.weatherForecast.period === 'afternoon') {
                return 'Afternoon';
            } else {
                return 'Evening';
            }
        }
    },

    methods: {
        refresh() {
            cmsService.getWeatherForecast().then((forecast) => {
                console.log(JSON.stringify(forecast));
                this.weatherForecast = forecast;
            })
        }
    }
}
</script>

<style scoped>
.weather {
    color: #c6c6c6;
}

.day-period {
    font-family: 'RobotoBold';
    font-size: 1.7em;
}

.temperature {
    color: #f1d104;
    font-size: 1.2em;
    text-align: center;
}

.felt-temperature {
}

.sky-and-rain {
    color: white;
    text-align: center;
    font-size: 1.1em;
}

.rain {
}

</style>
