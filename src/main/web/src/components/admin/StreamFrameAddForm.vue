<template>
    <b-form @submit.prevent="insertImage">
        <b-form-group id="imageFileGroup"
                      label="File" label-for="imageFileInput">
            <b-form-file id="imageFileInput"
                         v-model="imageFile"
                         plain
                         choose-label="Choose an image"
                         placeholder="No file chosen"
                         accept=".jpg, .png, .gif, .tiff">
            </b-form-file>
        </b-form-group>

        <div class="centered-btn-wrapper">
            <b-button type="submit" class="import-btn"
                      :disabled="busy || !imageFile"
                      variant="primary">
                Add <i v-if="busy" class="fa fa-spinner fa-spin"></i>
            </b-button>
        </div>
    </b-form>
</template>

<script>
import { cmsService } from '@/services/CmsService.js';

export default {
    props: ['streamId', 'newFrameIndex'],

    data() {
        return {
            busy: false,
            imageFile: null
        };
    },

    methods: {
        insertImage() {
            if (!this.imageFile) {
                return;
            }

            this.busy = true;
            cmsService.addStreamFrame(this.streamId, this.newFrameIndex, this.imageFile)
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
