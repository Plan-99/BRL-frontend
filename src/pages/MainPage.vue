<template>
    <q-page class="row">
        <div class="col-2 column">
            <div class="col-2 q-pa-md row">
                <div class="col text-center">
                    <q-icon name="power_settings_new" size="xl"></q-icon>
                    <div>종료</div>
                </div>
                <div class="col text-center">
                    <q-icon name="settings" size="xl"></q-icon>
                    <div>로봇관리</div>
                </div>
            </div>
            <div class="col-4 q-pa-md text-center">
                <div class="text-h6 text-bold">
                    작업 정보
                </div>
                <q-card flat bordered class="my-card">
                    <q-card-section class="text-bold">
                        <div>수확량</div>
                        <div class="text-primary">23개</div>
                    </q-card-section>

                    <q-separator inset />

                    <q-card-section class="text-bold">
                        <div>소요시간</div>
                        <div class="text-primary">00시간 17분</div>
                    </q-card-section>
                </q-card>
            </div>
            <div class="col q-pa-md text-center">
                <div class="text-h6 text-bold">
                    조작 방법
                </div>
                <div class="q-pt-sm q-gutter-y-sm">
                    <q-btn 
                        v-for="mode in ctr_modes"
                        :key="mode.value"
                        class="full-width" 
                        :color="ctr_mode === mode.value ? 'primary' : 'secondary'" 
                        :text-color="ctr_mode === mode.value ? 'white' : 'black'" 
                        size="lg" 
                        @click="ctr_mode = mode.value"
                    >{{ mode.label }}</q-btn>
                </div>
            </div>
        </div>
        <div class="col column q-pa-md">
            <div class="col-1">
                <div class="bg-white text-h6 text-bold q-pa-sm" style="border: 3px solid #02542D">
                <q-chip color="primary" text-color="white">대기중</q-chip>
                원하시는 작업을 위해 버튼을 눌러주세요!</div>
            </div>

            <div class="col">
                <canvas ref="rosCamera1"></canvas>
            </div>
        </div>
    </q-page>
</template>

<script setup>

import { ref, onMounted } from 'vue';
import ROSLIB from 'roslib';

const ctr_mode = ref('btn')

const ctr_modes = [
    { 'label': '누르기', icon: '', value: 'btn' },
    { 'label': '말하기', icon: '', value: 'voice' },
    { 'label': '손짓하기', icon: '', value: 'gesture' },
    { 'label': '컨트롤러 사용하기', icon: '', value: 'controller' },
]

const rosCamera1 = ref(null)
onMounted(() => {
    const ros = new ROSLIB.Ros({
        url : 'ws://localhost:9090'
    });

    ros.on('connection', function() {
        console.log('Connected to websocket server.');
    });

    ros.on('error', function(error) {
        console.log('Error connecting to websocket server: ', error);
    });

    ros.on('close', function() {
        console.log('Connection to websocket server closed.');
    });

    // 특정 토픽 구독 예시
    const cam1Listener = new ROSLIB.Topic({
        ros : ros,
        name : '/camera1/color/image_raw',
        messageType : 'sensor_msgs/Image'
    });

    // const cam2Listener = new ROSLIB.Topic({
    //     ros : ros,
    //     name : '/camera2/color/image_raw',
    //     messageType : 'sensor_msgs/Image'
    // });

    function base64ToArrayBuffer(base64) {
        const binaryString = window.atob(base64);  // base64 디코딩
        const len = binaryString.length;
        const bytes = new Uint8Array(len);  // 바이너리 데이터 저장할 배열 생성
        for (let i = 0; i < len; i++) {
            bytes[i] = binaryString.charCodeAt(i);
        }
        return bytes.buffer;  // ArrayBuffer 반환
    }

    function drawImageFromRGBArray(rgbArray, width, height) {
        const canvas = rosCamera1.value;
        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext('2d');

        // 이미지 데이터를 위한 빈 버퍼 생성
        const imageData = ctx.createImageData(width, height);
        const data = imageData.data;
        
        // sensor_msgs/Image의 data 필드는 바이너리 데이터 배열이므로 3개씩 읽어들임
        for (let i = 0, j = 0; i < rgbArray.length; i += 3, j += 4) {
            data[j] = rgbArray[i];        // R 값
            data[j + 1] = rgbArray[i + 1]; // G 값
            data[j + 2] = rgbArray[i + 2]; // B 값
            data[j + 3] = 255;            // A 값 (불투명도)
        }
        ctx.putImageData(imageData, 0, 0);

        // 캔버스를 HTML에 추가하여 표시
    }

    cam1Listener.subscribe(function(message) {
        if (message.header.seq % 3 === 0) {
            const decodedArrayBuffer = base64ToArrayBuffer(message.data);
            const unit8Array = new Uint8Array(decodedArrayBuffer);
            drawImageFromRGBArray(unit8Array, message.width, message.height)
        }
    });
})




</script>
  