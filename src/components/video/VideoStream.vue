<template>
    <video :src="track" autoplay="1" ref="videoRef" class="video">

    </video>
</template>

<script>
import {toRef, ref, watchEffect, onBeforeUnmount} from 'vue';
export default {
    name: 'VideoStream',
    props:{
        track: {
            type: [String, Object, Array]
        }
    },
    setup(props){
        const videoRef = ref(null);
        const track = toRef(props, 'track');
        watchEffect( ()=>{
           const element = videoRef.value;
           track.value.attach(element);
        })

        onBeforeUnmount(()=> {
           const element = videoRef.value;
           track.value.detach(element);
        })
        return {
            videoRef
        }
    }
}
</script>


<style scoped>
.video {
    height: 180px;
    width: 320px;
}
</style>