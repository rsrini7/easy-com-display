<template>
    <b-form @submit.prevent="addStream">
        <b-form-group id="streamNameGroup"
                      label="Name" label-for="streamNameInput">
            <b-form-input id="streamNameInput"
                          v-model="streamName"
                          type="text"
                          placeholder="Enter the name of the ad feed">
            </b-form-input>
        </b-form-group>

        <div class="centered-btn-wrapper">
            <b-button type="submit" class="import-btn"
                      :disabled="busy || !streamName"
                      variant="primary">
                Add <i v-if="busy" class="fa fa-spinner fa-spin"></i>
            </b-button>
        </div>
    </b-form>
</template>

<script>
import { cmsService } from '@/services/CmsService.js';

export default {
    data() {
        return {
            busy: false,
            streamName: null
        };
    },

    methods: {
        addStream() {
            this.busy = true;
            cmsService.addStream(this.streamName)
                .then((result) => {
                	console.print(result);
                    this.busy = false;
                });
        }
    }
}
</script>

<style scoped>
.centered-btn-wrapper {
   text-align: center;
}
</style>
