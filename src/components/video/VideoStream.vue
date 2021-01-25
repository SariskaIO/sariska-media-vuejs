<template>
    <video autoplay="1" ref="videoRef" class="video" />
</template>

<script>
import { toRef, watch } from 'vue';
export default {
    name: 'VideoStream',
    props:{
        track: String
    },
    setup(props){
        const videoRef = ref(null);
        const track = toRef(props, 'track');

        watch(()=>{
            window.addEventListener('load', ()=>{
                const element = videoRef.currrent;
                track.attach(element);
                return ()=>{
                    track.detach(element);
                }
            })
        })
    }
    
}
</script>


<style scoped>
.video {
    height: 180px;
    width: 320px;
}
</style>