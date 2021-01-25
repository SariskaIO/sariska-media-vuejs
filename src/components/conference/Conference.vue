<template>
    <div>
        <LocalStream :room="room" />
        <RemoteStream :room="room" />
    </div>
</template>

<script>
import { onBeforeUnmount, toRef, watch } from 'vue';
import JitsiMeetJS from "collabor8-media-transport";
import {conferenceConfig} from '../../constants/_constants';
import LocalStream from '../localStream/LocalStream';
import RemoteStream from '../remoteStream/RemoteStream';

export default {
    name: 'Conference',
    components: {
        LocalStream,
        RemoteStream
    },
    props: {
        connection: String
    },
    setup(props){
        const room = ref(null);

        function setRoom(val){
            room.value=val
        }
    watch(()=>{
        JitsiMeetJS.createLocalTracks({devices:["audio", "video"], resolution: "180"}).
        then(tracks => {
            setLocalTracks(tracks);
            room.value && tracks.forEach(track=>room.addTrack(track).catch(err =>console.log("track already added")));
        }).
        catch(()=>console.log("failed to fetch tracks"));
    })
        watch(()=>{
            const connection = toRef(props, 'connection');
            if ( connection.value === null ) {
            return;
            }
            const room = connection.initJitsiConference(conferenceConfig);

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
            setRemoteTracks(remoteTracks => [...remoteTracks.value, track]);
        }

        room.on(JitsiMeetJS.events.conference.CONFERENCE_JOINED, onConferenceJoined);
        room.on(JitsiMeetJS.events.conference.TRACK_ADDED, onRemoteTrack);
        room.on(JitsiMeetJS.events.conference.TRACK_REMOVED, onTrackRemoved);
        room.join();

        })
        onBeforeUnmount(()=>{
        function beforeUnload(event) {
            if (room && room.isJoined()) {
                room.leave().then(() => connection.disconnect(event));
            }
        }
            window.addEventListener('beforeunload', beforeUnload);
            
            beforeUnload();

        })
        return {
            room
        }
    }
    
}
</script>