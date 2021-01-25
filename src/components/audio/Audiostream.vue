<template>
    <audio autoplay="1" ref="audioRef" />
</template>

<script>
import {toRef, ref, onMounted, onUnmounted} from 'vue';
export default {
    name: 'AudioStream',
    props:{
        track: String
    },
    setup(props){
        const audioRef = ref(null);
        const track = toRef(props, 'track');

        onMounted( ()=>{
          const element = audioRef.value.current;
          track.value.attach(element);
        })

        onUnmounted(()=> {
          const element = audioRef.value.current;
          track.value.detach(element);

        })
    }
}
</script>