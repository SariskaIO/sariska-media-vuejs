<template>
    <video autoplay="1" ref="videoRef" class="video" />
</template>

<script>
import {toRef, ref, onMounted, onUnmounted} from 'vue';
export default {
    name: 'VideoStream',
    props:{
        track: String
    },
    setup(props){
        const videoRef = ref(null);
        const track = toRef(props, 'track');
        onMounted( ()=>{
           const element = videoRef.value.current;
           track.value.attach(element);
        })

        onUnmounted(()=> {
           const element = videoRef.value.current;
           track.value.detach(element);
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