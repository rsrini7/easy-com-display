<template>
<div class="admin-stream-list">
    <h3>Ad feeds</h3>

    <b-link v-b-modal.streamAddModal>
        <span><i class="fa fa-plus-square"></i>&nbsp;Create an ad feed</span>
    </b-link>
    &nbsp;&nbsp;
    <b-link v-b-modal.presentationImportModal>
        <span><i class="fa fa-cloud-download"></i>&nbsp;Import a presentation</span>
    </b-link>

    <div v-if="streamList">
        <table class="admin-stream-table">
            <tbody>
                <tr v-for="stream in orderedStreams" class="admin-stream-row">
                    <td class="admin-stream-playlist-col">
                        <span v-if="playlist && playlist.streamsMap[stream.id]"
                              class="admin-stream-playlist-input"
                              @click="removeFromPlaylist(stream)">
                            <i class="fa fa-star"></i>
                        </span>
                        <span v-if="playlist && !playlist.streamsMap[stream.id]"
                              class="admin-stream-playlist-input"
                              @click="addToPlaylist(stream)">
                            <i class="fa fa-star-o"></i>
                        </span>
                    </td>
                    <td class="admin-stream-thumbnail-col">
                        <router-link :to="{ name: 'stream-detail', params: { id: stream.id }}">
                            <img v-if="stream.frames.length > 0" :src="cmsService.getFrameThumbnailSource(stream, 0)">
                            <span v-if="stream.frames.length == 0" class="empty-stream-thumbnail"><i class="fa fa-folder fa-5x"></i></span>
                        </router-link>
                    </td>
                    <td class="admin-stream-name-col"  @click="goToStreamDetail(stream)">
                        <div class="admin-stream-name">
                            {{stream.name}}
                        </div>
                    </td>
                    <td class="admin-stream-last-update-col">
                        <div class="admin-stream-last-update">
                            Updated {{getLastUpdateDate(stream)}}
                        </div>
                    </td>
                    <td class="admin-stream-delete-col">
                        <span class="admin-stream-delete-input" @click="showStreamDeletionConfirmModal(stream)"><i class="fa fa-trash"></i></span>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <empty-content v-if="!streamList || orderedStreams.length == 0"
        message="You do not have an ad feed">
    </empty-content>

    <!-- Modal to add an ad feed -->
    <b-modal id="streamAddModal"
             ref="streamAddModal"
             title="Create an ad feed"
             size="lg">
        <stream-add-form></stream-add-form>
        <div slot="modal-footer">
            <b-btn variant="secondary" @click="closeStreamAddModal">
                Close
            </b-btn>
        </div>
    </b-modal>

    <!-- Modal to import a presentation -->
    <b-modal id="presentationImportModal"
             ref="presentationImportModal"
             title="Import a presentation"
             size="lg">
        <presentation-import-form></presentation-import-form>
        <div slot="modal-footer">
            <b-btn variant="secondary" @click="closePresentationImportModal">
                Close
            </b-btn>
        </div>
    </b-modal>

    <!-- Modal pour supprimer un flux d'annonces -->
    <b-modal id="streamDeletionConfirmModal"
             ref="streamDeletionConfirmModal"
             title="Suppression d'un fil d'annonces">
        <span v-if="streamToDelete" >Voulez-vous supprimer "{{streamToDelete.name}}" ?</span>

        <div slot="modal-footer">
            <b-button variant="danger"
                      @click="confirmStreamDeletion"
                      href="">
                <i class="fa fa-trash"></i> Oui, supprimer
            </b-button>
            <b-button variant="secondary"
                      @click="closeStreamDeletionConfirmModal"
                      href="">
                Non, annuler
            </b-button>
        </div>
    </b-modal>
</div>
</template>

<script>
import ld from 'lodash'

import { EventBus } from '@/services/EventBus.js';

import { cmsService } from '@/services/CmsService.js';
import { playlistService } from '@/services/PlaylistService.js';

import StreamAddForm from '@/components/admin/StreamAddForm.vue';
import PresentationImportForm from '@/components/admin/PresentationImportForm.vue';
import EmptyContent from '@/components/admin/EmptyContent.vue';

export default {
    components: {
        'stream-add-form': StreamAddForm,
        'presentation-import-form': PresentationImportForm,
        'empty-content': EmptyContent
    },

    data() {
        return {
            cmsService,
            streamList: null,
            playlist: null,
            streamToDelete: null
        };
    },

    mounted() {
        this.getStreamList();
        this.getPlaylist();
        EventBus.$on('newStreamEvent', this.onNewStream);
    },

    beforeDestroy() {
        EventBus.$off('newStreamEvent', this.onNewStream);
    },

    computed: {
        orderedStreams: function() {
            return ld.orderBy(this.streamList, 'name');
        }
    },

    methods: {
        getStreamList() {
            return cmsService.getStreamList().then((streamList) => {
                this.streamList = streamList;
            });
        },

        getPlaylist() {
            return playlistService.getPlaylist().then((playlist) => {
                this.playlist = playlist;
            });
        },

        removeFromPlaylist(stream) {
            return playlistService.removeStreamFromPlaylist(stream).then((playlist) => {
                this.playlist.streams = playlist.streams;
                this.playlist.streamsMap = playlist.streamsMap;
            });
        },

        addToPlaylist(stream) {
            return playlistService.addStreamToPlaylist(stream).then((playlist) => {
                this.playlist.streams = playlist.streams;
                this.playlist.streamsMap = playlist.streamsMap;
            });
        },

        goToStreamDetail(stream) {
            this.$router.push({name: 'stream-detail', params: { id: stream.id }});
        },

        showStreamDeletionConfirmModal(stream) {
            this.streamToDelete = stream;
            this.$refs.streamDeletionConfirmModal.show();
        },

        confirmStreamDeletion() {
            this.deleteStream().then(() => this.closeStreamDeletionConfirmModal());
        },

        deleteStream() {
            return cmsService.deleteStream(this.streamToDelete.id).then(() => {
                return this.getStreamList()
            });
        },

        getLastUpdateDate(stream) {
            let lastUpdateDate = new Date(stream.lastUpdateDateTime);
            let day = lastUpdateDate.getDate();
            let month = lastUpdateDate.getMonth() + 1;
            let year = lastUpdateDate.getFullYear();
            let hours = lastUpdateDate.getHours();
            let minutes = lastUpdateDate.getMinutes();
            if (minutes < 10) {
                minutes = '0' + minutes;
            }
            return day + '/' + month + '/' + year + ' à ' + hours + ':' + minutes;
        },

        onNewStream() {
            this.closeStreamAddModal();
            this.closePresentationImportModal();
            this.getStreamList();
        },

        closeStreamAddModal() {
            this.$refs.streamAddModal.hide();
        },

        closePresentationImportModal() {
            this.$refs.presentationImportModal.hide();
        },

        closeStreamDeletionConfirmModal() {
            this.$refs.streamDeletionConfirmModal.hide();
            this.streamToDelete = null;
        }
    }
}
</script>

<style scoped>
.admin-stream-table {
    table-layout: auto;
    margin-top: 16px;
    border: 1px solid rgba(0, 0, 0, 0.2);
    width: 100%;
}

.admin-stream-row {
    background-color: white;
    box-shadow: 1px 4px 6px rgba(0, 0, 0, .3);
    border: solid 1px rgba(0, 0, 0, 0.1);
}

.admin-stream-row:hover {
    background-color: rgba(0, 123, 255, 0.6);
    color: white;
}

.admin-stream-playlist-col {
    padding: 8px;
    text-align: center;
    font-size: 1.5em;
    width: 60px;
}

.admin-stream-playlist-input {
    color: #E55934;
}

.admin-stream-playlist-input:hover {
    cursor: pointer;
}

.admin-stream-thumbnail-col {
    padding: 8px;
    text-align: center;
}

.admin-stream-name-col {
    position: relative;
    width: 60%;
    padding-left: 8px;
}

.admin-stream-name-col:hover {
    cursor: pointer;
}

.admin-stream-name {
    font-size: 1.2em;
}

.admin-stream-last-update-col {
    width: auto;
}

.admin-stream-last-update {
    font-size: 0.9em;
    font-style:italic;
}

.admin-stream-delete-col {
    padding: 4px;
    text-align: center;
    font-size: 1.5em;
    background-color: rgba(0,0,0,0.1);
    color: rgba(0,0,0,0.5);
    width: 40px;
}

.admin-stream-delete-input:hover {
    color: rgba(255, 0, 0, 0.7);
    cursor: pointer;
}

.empty-stream-thumbnail {
    color: #A9A9A9;
    vertical-align: middle;
}
</style>
