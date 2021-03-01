<template>
    <div>
        <div  v-for="(track, index) in localTracks" :key="index">
            <div v-if="track.isVideoTrack()">
                <VideoStream :track="track" />
            </div>
            <div v-else>
                <AudioStream :track="track" />
            </div>
        </div>
    </div>
</template>

<script>
import { toRef, watch } from 'vue';
import AudioStream from '../audio/Audiostream';
import VideoStream from '../video/VideoStream';

export default {
    name: 'LocalStream',
    props: {
        localTracks: Array
    },
    components: {
        AudioStream,
        VideoStream
    },
    setup(props){
        const lTracks=toRef(props, 'localTracks');
        watch(()=>{
        console.log('local1', lTracks.value);

        })
    }
}
</script>
