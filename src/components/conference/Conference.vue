<template>
    <div>
        <LocalStream :localTracks="localTracks" />
        <RemoteStream :remoteTracks="remoteTracks" />
        <div>i am {{localTracks}}</div>
        <div>me am {{remoteTracks}}</div>
    </div>
</template>

<script>
import { toRef, ref, watch, onUnmounted } from 'vue';
import JitsiMeetJS from "sariska-media-transport";
import {conferenceConfig} from '@/constants';
import LocalStream from '../localStream/LocalStream';
import RemoteStream from '../remoteStream/RemoteStream';

export default {
    name: 'Conference',
    components: {
        LocalStream,
        RemoteStream
    },

    props: {
        connection: Object
    },

    setup(props){

        const room = ref(null);
        const remoteTracks = ref([]);
        const localTracks = ref([]);
        const connection = toRef(props, 'connection');

        const setRoom = (roomValue) => {
            room.value = roomValue
        }
        const setRemoteTracks = (trackValue) => remoteTracks.value = [...remoteTracks.value, trackValue];
        
        const setLocalTracks = (trackValue) => localTracks.value = [...localTracks.value, trackValue];   

        watch(room, ()=>{
            JitsiMeetJS.createLocalTracks({devices:["audio", "video"], resolution: "180"}).
            then(tracks => {
            setLocalTracks(tracks);
            room.value && tracks.forEach(track=>room.value.addTrack(track).catch(err =>console.log("track already added", err)));
        }).
        catch(()=>console.log("failed to fetch tracks"));
        })

        watch(connection, ()=>{
            if (!connection.value) {
                return; 
            }

            const room = connection.value.initJitsiConference(conferenceConfig);

            const onConferenceJoined = ()=> {
                setRoom(room);
            }
            const onTrackRemoved = (track)=> {
                setRemoteTracks(remoteTracks.value.filter(item => item.track.id !== track.track.id));
            }
    
            const onRemoteTrack = (track)=> {
                if (!track  || track.isLocal()) {
                    return;
                }
                setRemoteTracks(track);
            }

            room.on(JitsiMeetJS.events.conference.CONFERENCE_JOINED, onConferenceJoined);
            room.on(JitsiMeetJS.events.conference.TRACK_ADDED, onRemoteTrack);
            room.on(JitsiMeetJS.events.conference.TRACK_REMOVED, onTrackRemoved);
            room.join();
        })

          onUnmounted(()=>{
             if(!connection.value) return;
            function beforeUnload(event) {
                if (room.value && room.value.isJoined()) {
                    room.value.leave().then(() => connection.value.disconnect(event));
                }
            }
            beforeUnload();
          })

        return {
            remoteTracks, localTracks
        }
    }
}
</script>