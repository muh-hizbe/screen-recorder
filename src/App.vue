<template>
    <div class="min-h-screen">
        <div class="animate-bounce border-t-4 border-blue-500 mb-5"></div>
        <div class="container mx-auto">
            <h1 class="text-3xl font-semibold text-center underline leading-loose"><span class="bg-white border-2 border-gray-700 p-3 rounded">🖥 Screen Media</span></h1>
            <p class="text-center font-semibold leading-loose">Screen recording</p>
            
            <!-- <div class="block md:hidden text-center leading-loose mt-10 text-2xl px-5 underline">Tidak mendukung di browser mobile</div> -->
            
            <div class="text-center mt-10" :class="stop ? 'grid grid-cols-2 gap-2 mx-96' : ''">
                <button v-if="stop" @click="startRecording()" class="start transition duration-300 ease-in-out font-semibold border-2 border-blue-500 px-3 py-2 rounded hover:bg-blue-500 hover:text-white">⏺ Start Recording</button>
                <button v-if="start" @click="stopRecording()" class="stop transition duration-300 ease-in-out font-semibold border-2 border-red-500 px-3 py-2 rounded hover:bg-red-500 hover:text-white">⏹ Stop Recording</button>
                <button v-if="stop" @click="() => { videoSrc = null }" class="stop transition duration-300 ease-in-out font-semibold border-2 border-red-500 px-3 py-2 rounded hover:bg-red-500 hover:text-white">🔴 Reset</button>
            </div>
            <div v-if="videoSrc" class="transition duration-300 text-center py-5">
                <a :href="videoSrc" download class="transition duration-300 ease-in-out animate-bounce font-semibold border-2 border-green-500 px-3 py-2 rounded hover:bg-green-500 hover:text-white">💾 Unduh Video</a>
            </div>
            
            <div v-if="start" class="text-center text-red-700 mb-5 leading-loose underline">Don't close or refresh this page while recording !</div>
            
            <div v-if="start" class="flex text-center justify-center">
                <div class="animate-bounce">🟢</div>
                <div class="animate-pulse"> Recording ...</div>
            </div>

            <Video v-if="videoSrc" :src="videoSrc" class="transition duration-300 my-5 text-center" />
        </div>
    </div>
</template>

<script>
import { onMounted, reactive, ref } from 'vue'
import Video from './components/Video.vue'
export default {
    name: 'App',

    components: {
        Video
    },

    setup() {
        const start = ref(false)
        const stop = ref(true)
        // const stream = ref({})
        // const recorder = ref({})
        const videoSrc = ref(null)
        const data = reactive({
            stream: {},
            recorder: {}
        })

        // onMounted(() => {
        //     // if (!navigator.getDisplayMedia && !navigator.mediaDevices.getDisplayMedia) {
        //     //     const msg = "Browser tidak mendukung getDisplayMedia API"
        //     //     console.log(msg);
        //     //     throw new Error(msg)
        //     // }
        //     if (Navigator.mediaDevices && Navigator.mediaDevices.getUserMedia) {
        //         console.log('getUserMedia supported.');
        //         navigator.mediaDevices.getUserMedia (
        //             // constraints - only audio needed for this app
        //             {
        //                 audio: true
        //             })

        //             // Success callback
        //             .then(function(stream) {
        //                 // console.log(stream);
        //             })

        //             // Error callback
        //             .catch(function(err) {
        //                 console.log('The following getUserMedia error occurred: ' + err);
        //             }
        //         );
        //     } else {
        //         console.log('getUserMedia not supported on your browser!');
        //     }
        // })

        onMounted(() => {
            if (!navigator.mediaDevices && !navigator.mediaDevices.getUserMedia()) {
                alert('Browser tidak mendukung untuk melakukan perekaman layar')
            }
            Notification.requestPermission().then(result => console.log(result))
        })

        const startRecording = async () => {
            start.value = true
            stop.value = false
            // console.log('starting recording');
            try {
                // stream.value = await navigator.mediaDevices.getDisplayMedia({
                //     video: { mediaSource: 'screen'}
                // })
                // stream.value = await navigator.mediaDevices.getDisplayMedia({
                
                data.stream = await navigator.mediaDevices.getDisplayMedia({
                    video: { cursor: "always" },
                    audio: {
                        echoCancellation: true,
                        noiseSuppression: true,
                        sampleRate: 44100
                    }
                })
                // recorder.value = new MediaRecorder(stream, { mimeType: "video/webm; codecs=vp9" })
                data.recorder = new MediaRecorder(data.stream, { mimeType: "video/webm; codecs=vp9" })
                // console.log(recorder);
    
                const chunks = []
                data.recorder.ondataavailable = (event) => chunks.push(event.data)
                data.recorder.onstop = e => {
                    // const completeBlob = new Blob(chunks, { type: chunks[0].type });
                    const completeBlob = new Blob(chunks, { type: "video/webm" });
                    // console.log(completeBlob);
                    videoSrc.value = URL.createObjectURL(completeBlob);
                };
                data.recorder.start();                
            } catch (error) {
                start.value = false
                stop.value = true
                alert(error)
            }
        }

        const stopRecording = () => {
            start.value = false
            stop.value = true

            data.recorder.stop()
            data.stream.getVideTracks()[0].stop()
            // console.log(data.stream);
            console.log('stop recording');
            
        }

        return {
            start, stop, videoSrc,
            // stream, recorder,
            data,
            startRecording, stopRecording
        }
    },

    // created() {
    //     if (!navigator.getDisplayMedia && !navigator.mediaDevices.getDisplayMedia) {
    //         const msg = "Browser tidak mendukung getDisplayMedia API"
    //         console.log(msg);
    //         throw new Error(msg)
    //     }
    // },

    // data() {
    //     return {
    //         start: false,
    //         stop: true,
    //         stream: null,
    //         recorder: null,
    //         videoSrc: null
    //     }
    // },

    // methods: {
    //     async startRecording() {
    //         this.start = true
    //         this.stop = false
    //         console.log('starting recording');
    //         this.stream = await navigator.mediaDevices.getDisplayMedia({
    //             video: { mediaSource: 'screen'}
    //         })
    //         this.recorder = new MediaRecorder(this.stream)

    //         const chunks = []
    //         this.recorder.ondataavailable = (event) => chunks.push(event.data)
    //         this.recorder.onstop = e => {
    //             const completeBlob = new Blob(chunks, { type: chunks[0].type });
    //             this.videoSrc = URL.createObjectURL(completeBlob);
    //         };
    //         this.recorder.start();
    //     },
    //     stopRecording() {
    //         this.start = false
    //         this.stop = true

    //         this.recorder.stop()
    //         this.stream.getVideTracks()[0].stop()
    //         console.log('stop recording');
            
    //     }
    // }
}
</script>
