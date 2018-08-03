<template>
    <b-form @submit.prevent="importPresentation">
        <b-form-group id="streamNameGroup"
                      label="Name" label-for="streamNameInput">
            <b-form-input id="streamNameInput"
                          v-model="streamName"
                          type="text"
                          placeholder="Enter the name of the ad feed">
            </b-form-input>
        </b-form-group>
        <b-form-group id="presentationFileGroup"
                      label="File" label-for="presentationFileInput">
            <b-form-file id="presentationFileInput"
                         v-model="presentationFile"
                         plain
                         choose-label="Choose a presentation"
                         placeholder="No file chosen">
            </b-form-file>
        </b-form-group>

        <div class="centered-btn-wrapper">
            <b-button type="submit" class="import-btn"
                      :disabled="busy || !presentationFile || !streamName"
                      variant="primary">
                Import <i v-if="busy" class="fa fa-spinner fa-spin"></i>
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
            presentationFile: null,
            streamName: null
        };
    },

    methods: {
        importPresentation() {
            if (!this.presentationFile) {
                return;
            }

            this.busy = true;
            cmsService.importPresentation(this.presentationFile, this.streamName)
                .then((result) => {
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
