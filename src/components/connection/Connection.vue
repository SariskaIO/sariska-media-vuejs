<template>
    <div>
        <Conference :connection="connection" />
    </div>
</template>

<script>
import Conference from "../conference/Conference.vue";
import JitsiMeetJS from 'collabor8-media-transport';
import {connectionConfig, initSDKConfig} from '../../constants/_constants';
import {getToken, getL} from '../../utils/index';
import { onMounted, onUnmounted, ref } from 'vue';

export default {
    name: 'Connection',
    components: {
        Conference
    },
    setup(){
        const connection = ref(null);

        function setConnection(val){
            connection.value = val;
        }

        onMounted(()=>{
            JitsiMeetJS.init(initSDKConfig);
        JitsiMeetJS.setLogLevel(JitsiMeetJS.logLevels.ERROR); //TRACE ,DEBUG, INFO, LOG, WARN, ERROR
        let conn;

        const fetchData =  async ()=>{
            const token = await getToken();
            if (!token) {
                return;
            }
            conn = new JitsiMeetJS.JitsiConnection(token, connectionConfig);
            conn.addEventListener(JitsiMeetJS.events.connection.CONNECTION_ESTABLISHED, onConnectionSuccess);
            conn.addEventListener(JitsiMeetJS.events.connection.CONNECTION_FAILED, onConnectionFailed);
            conn.addEventListener(JitsiMeetJS.events.connection.CONNECTION_DISCONNECTED, onConnectionDisconnected);
            conn.addEventListener(JitsiMeetJS.events.connection.PASSWORD_REQUIRED, onConnectionDisconnected);
            conn.connect();
        }

        const onConnectionSuccess = ()=>{
            setConnection(conn);
        }

        const onConnectionDisconnected = (error)=>{
            console.log('connection disconnect!!!', error);
            if (!connection) {
                return;
            }
            connection.value.removeEventListener(
                JitsiMeetJS.events.connection.CONNECTION_ESTABLISHED,
                onConnectionSuccess);
            connection.value.removeEventListener(
                JitsiMeetJS.events.connection.CONNECTION_FAILED,
                onConnectionFailed);
            connection.value.removeEventListener(
                JitsiMeetJS.events.connection.CONNECTION_DISCONNECTED,
                onConnectionDisconnected);

        }

        const onConnectionFailed = async (error)=> {
            console.log('connection failed!!!', error);
            if (error === "connection.passwordRequired") {  // token expired,  fetch new token and set again
                const token = await getToken();
                conn.setToken(token);
            }
        }

        const updateNetwork = ()=>{  //  set internet connectivity status
            JitsiMeetJS.setNetworkInfo({isOnline: window.navigator.onLine});
        }


        window.addEventListener("offline", updateNetwork);
        window.addEventListener("online", updateNetwork);

        fetchData();
        return () => {
            connection.disconnect();
        }
            })

        onUnmounted(()=>{
            window.removeEventListener("offline", updateNetwork);
            window.removeEventListener("online", updateNetwork);
        })
        
    return {
        connection
    }
    }
    
}
</script>