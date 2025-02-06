<template>
    <q-page class="row q-mx-lg">
        <div class="col-2">
            <div class="q-pa-md row">
                <div class="col text-center" @click="showQuitRobotDialog = true">
                    <q-icon name="power_settings_new" size="xl"></q-icon>
                    <div>종료</div>
                </div>
                <div class="col text-center">
                    <q-icon name="settings" size="xl"></q-icon>
                    <div>로봇관리</div>
                </div>
            </div>
            <div class="q-pa-md text-center">
                <div class="text-h6 text-bold">
                    작업 정보
                </div>
                <q-card flat bordered class="my-card">
                    <q-card-section class="text-bold text-h6">
                        <div>수확량</div>
                        <div class="text-primary">23개</div>
                    </q-card-section>

                    <q-separator inset />

                    <q-card-section class="text-bold text-h6">
                        <div>소요시간</div>
                        <div class="text-primary">00시간 17분</div>
                    </q-card-section>
                </q-card>
            </div>
            <div class="q-pa-md text-center">
                <div class="text-h6 text-bold">
                    조작 방법
                    <q-icon name="help_outline" size="md">
                        <q-popup-proxy>
                            <div class="q-pa-md text-center ">
                                <div class="text-h6">조작 방법 도우미</div>
                                <div class="q-mt-md" style="white-space: pre;">
                                    1. ‘말하기 버튼‘을 누르시면, <br>
                                    말하기로 조작을 할 수 있습니다. <br><br>
                                    2. ‘손짓하기 버튼‘을 누르시면, <br>
                                    손짓하기로 조작을 할 수 있습니다. <br><br>
                                    3. ‘컨트롤러 사용하기 버튼’을 누르시면, <br>
                                    컨트롤러로 조작을 할 수 있습니다. <br>
                                </div>
                            </div>
                        </q-popup-proxy>
                    </q-icon>
                </div>
                <div class="q-pt-sm q-gutter-y-sm">
                    <q-btn 
                        v-for="mode in ctr_modes"
                        :key="mode.value"
                        class="full-width" 
                        :color="ctr_mode === mode.value ? 'primary' : 'green-2'" 
                        :text-color="ctr_mode === mode.value ? 'white' : 'black'" 
                        size="lg" 
                        @click="ctr_mode = ctr_mode !== mode.value ? mode.value : null"
                        style="white-space: pre; height: 80px"
                    >
                        <q-icon :name="mode.icon" class="q-mr-lg"></q-icon>
                        <div>{{ mode.label }}</div>
                    </q-btn>
                </div>
            </div>
        </div>
        <div class="col q-pa-md q-px-xl">
            <div class="col-1">
                <div class="bg-white text-h6 text-bold q-pa-sm" style="border: 3px solid #02542D;" >
                <q-chip :color="status.color" text-color="black">{{ status.label }}</q-chip>
                {{ status.msg }}</div>
            </div>

            <div class="q-pt-lg row">
                <div class="col-8">
                    <div style="position: relative;">
                        <!-- <canvas ref="rosCameraMain" style=" width: 100%; aspect-ratio: 4 / 3;" :style="status.value === 'finding_missed_tomato' ? 'cursor: pointer; border: 6px solid #0000FF': 'border: 3px solid #02542D;'" @click="videoClicked"></canvas> -->
                        <video 
                            ref="rosCameraMain" 
                            autoplay playsinline muted 
                            style=" width: 100%; aspect-ratio: 4 / 3; background-color: black;"
                            :style="status.value === 'finding_missed_tomato' ? 'cursor: pointer; border: 6px solid #0000FF': 'border: 3px solid #02542D;'"
                            @click="videoClicked"
                        ></video>
                    </div>
                </div>
                <div class="col column">
                    <div style="position: relative;">
                        <q-chip class="absolute-top-right q-ma-md" color="primary" text-color="white" icon="refresh" clickable=""
                            @click="switchCam" style="z-index: 10;"
                        >화면 바꾸기</q-chip>
                        <!-- <canvas ref="rosCameraSub" style="border: 3px solid #02542D; border-left: none; width: 100%; aspect-ratio: 4 / 3;"></canvas> -->
                        <video ref="rosCameraSub" 
                            autoplay playsinline muted 
                            style="border: 3px solid #02542D; border-left: none; width: 100%; aspect-ratio: 4 / 3; background-color: black;"
                        ></video>
                    </div>
                    <div class="text-center q-mt-xl">

                        <q-btn color="red" size="xl" @click="emergency_stop">
                            <div class="column">
                                <q-icon center size="5em" name="warning" />
                                <div>긴급정지</div>
                            </div>
                        </q-btn>
                    </div>
                </div>
            </div>
            <div class="row q-col-gutter-x-xl">
                <div
                    v-for="btn in robot_btns"
                    :key="btn.value"
                    class="col"
                >
                    <q-btn
                        color="secondary"
                        size="lg"
                        text-color="black"
                        :disable="!status.availabe_btns.includes(btn.value)"
                        @click="clickRobotBtn(btn.value)"
                    >
                        <div class="column text-center">
                            <q-icon center size="4em" :name="btn.icon" :color="status.availabe_btns.includes(btn.value) ? btn.color : 'grey-6'" />
                            <div>{{ btn.label }}</div>
                        </div>
                    </q-btn>
                </div>
                <div class="col-4"></div>
            </div>
        </div>
        <div class="fixed-bottom-right q-ma-lg q-pa-xl text-center text-white" style="background-color: #000000aa;" v-if="dialog">
            <div class="row q-col-gutter-x-md" v-if="dialog === 'select_correction'">
                <div class="q-mt-xl">
                    <q-icon class="cursor-pointer" style="vertical-align: center;" name="cancel" size="3em" @click="closeDialog"></q-icon>
                </div>
                <div class="text-h5 row q-mt-xl">도움이 필요하신가요?</div>
                <div
                    v-for="btn in correction_btns"
                    :key="btn.value"
                >
                    <q-btn
                        color="primary"
                        size="lg"
                        text-color="white"
                        @click="btn.clicked"
                    >
                        <div class="text-center">
                            
                            <q-icon size="4em" :name="btn.icon" />
                            <div style="white-space: pre;">{{ btn.label }}</div>
                        </div>
                    </q-btn>
                </div>
            </div>
            <div v-if="boolean_dialog_list.map((e) => e.id).includes(dialog)" class="row q-col-gutter-x-lg" >
                <div class="q-mt-xl">
                    <q-icon class="cursor-pointer" style="vertical-align: center;" name="cancel" size="3em" @click="closeDialog"></q-icon>
                </div>
                <div class="text-h5 q-mt-xl">{{ boolean_dialog_list.find((e) => e.id === dialog).msg }}</div>
                <div class="">
                    <q-btn
                        color="primary"
                        size="xl"
                        text-color="white"
                        @click="boolean_dialog_list.find((e) => e.id === dialog).onOk"
                    >
                        <div class="text-center">
                            <q-icon size="4em" name="circle" />
                            <div>예</div>
                        </div>
                    </q-btn>
                </div>
                <div class="">
                    <q-btn
                        color="primary"
                        size="xl"
                        text-color="white"
                        @click="boolean_dialog_list.find((e) => e.id === dialog).onNo"
                    >
                        <div class="text-center">
                            <q-icon size="4em" name="close" />
                            <div>아니오</div>
                        </div>
                    </q-btn>
                </div>
            </div>
            <div v-if="notice_dialog_list.map((e) => e.id).includes(dialog)" class="row q-col-gutter-x-lg" >
                <div>
                    <q-icon style="vertical-align: center;" name="check" size="3em"></q-icon>
                </div>
                <div class="text-h5">{{ notice_dialog_list.find((e) => e.id === dialog).msg }}</div>
            </div>
            <div v-if="order_dialog_list.map((e) => e.id).includes(dialog)" class="row q-col-gutter-x-lg" >
                <div>
                    <q-icon style="vertical-align: center;" name="cancel" size="3em" @click="closeDialog"></q-icon>
                </div>
                <div class="text-h5">{{ order_dialog_list.find((e) => e.id === dialog).msg }}</div>
            </div>
        </div>
        <q-dialog v-model="showQuitRobotDialog" persistent>
            <div v-if="quitRobotStatus === 'ask'">
                <div class="text-white text-h4 q-mb-lg">로봇을 종료할까요?</div>
                <div class="text-center q-gutter-x-lg">
                    <q-btn color="blue" text-color="white" size="lg" style="width: 100px" @click="quitRobot">예</q-btn>
                    <q-btn color="white" text-color="black" size="lg" @click="showQuitRobotDialog = false" style="width: 100px">아니오</q-btn>
                </div>
            </div>
            <div v-else-if="quitRobotStatus === 'quitting'">
                <div class="text-white text-h4 q-mb-lg">로봇이 복귀한 후 자동으로 종료됩니다.</div>
            </div>
            <div v-else-if="quitRobotStatus === 'quitted'">
                <div class="text-white text-h4 q-mb-lg">로봇이 성공적으로 종료되었습니다.</div>
            </div>
        </q-dialog>
    </q-page>
</template>

<script setup>

import { ref, onMounted } from 'vue';
import { useQuasar } from 'quasar'
import { useRouter } from 'vue-router'


const $q = useQuasar()
const $router = useRouter() 
const ctr_mode = ref(null)

const ctr_modes = [
    { 'label': '말하기', icon: 'mic', value: 'voice' },
    { 'label': '손짓하기', icon: 'waving_hand', value: 'gesture' },
    { 'label': '컨트롤러\n사용하기', icon: 'sports_esports', value: 'controller' },
]

const robot_btns = [
    { 'label': '작업시작', icon: 'play_arrow', value: 'start', color: 'primary', active_status: [''] },
    { 'label': '작업수정', icon: 'build', value: 'correct', color: 'red'  },
    { 'label': '일시정지', icon: 'stop', value: 'stop', color: 'orange'  },
    { 'label': '작업완료', icon: 'done_outline', value: 'finish', color: 'blue'  },
]

const correction_btns = [
    { 'label': '수확할\n토마토 확인', icon: 'img:icons/1.png', value: 'target_check', clicked: () => { dialog.value = 'harvest'; } },
    { 'label': '수확후\n토마토 확인', icon: 'img:icons/2.png', value: 'quality_check', clicked: () => { dialog.value = 'discard'; } },
    { 'label': '놓친 토마토\n확인', icon: 'img:icons/3.png', value: 'missed_check', clicked: () => { dialog.value = 'missed_tomato'; } },
]

const rosCameraMain = ref(null)
const rosCameraSub = ref(null)

const currentMain = ref('cam1')

const status_list = [
    { label: '대기중', value: 'pending', msg: '작업을 시작하려면 [작업 시작]을 눌러주세요!', color: 'green-2', availabe_btns: ['start'] },
    { label: '작업진행', value: 'moving', msg: '작업을 위해 이동중이에요!', color: 'yellow-2', availabe_btns: ['stop', 'finish'] },
    { label: '작업진행', value: 'harvesting', msg: '토마토를 수확중이에요!', color: 'yellow-2', availabe_btns: ['correct', 'stop', 'finish'] },
    { label: '작업진행', value: 'pass_harvesting', msg: '토마토를 수확중이에요!', color: 'yellow-2', availabe_btns: ['correct', 'stop', 'finish'] },
    { label: '작업진행', value: 'discarding', msg: '토마토를 폐기중이에요!', color: 'yellow-2', availabe_btns: ['correct', 'stop', 'finish'] },
    { label: '작업진행', value: 'collecting', msg: '토마토를 수확중이에요!', color: 'yellow-2', availabe_btns: ['correct', 'stop', 'finish'] },
    { label: '작업중지', value: 'paused', msg: '작업을 계속 하려면 [작업 시작]을 눌러주세요!', color: 'red-2', availabe_btns: ['start'] },
    { label: '오류발생', value: 'correcting', msg: '토마토 수확 중 문제가 발생했나요?', color: 'red-2', availabe_btns: [] },
    { label: '오류발생', value: 'finding_missed_tomato', msg: '토마토 수확 중 문제가 발생했나요?', color: 'red-2', availabe_btns: [] },
    { label: '작업완료', value: 'finished', msg: '모든 작업을 완료했습니다! 추가 작업이 있다면 알려주세요!', color: 'purple-2', availabe_btns: ['start'] },
]
const status = ref(status_list.find((e) => e.value === 'pending'))

const dialog = ref(null)

function switchCam() {
    let tmpSrcObject = null
    tmpSrcObject = rosCameraMain.value.srcObject
    rosCameraMain.value.srcObject = rosCameraSub.value.srcObject
    rosCameraSub.value.srcObject = tmpSrcObject

    if (currentMain.value === 'cam1') {
        currentMain.value = 'cam2'
    } else {
        currentMain.value = 'cam1'
    }
}

const boolean_dialog_list = [
    { id: 'finish', msg: '작업을 완료할까요?', onOk: () => { statusBuffer.value = status.value; status.value = status_list.find((e) => e.value === 'finished'); dialog.value = null; }, onNo: closeDialog },
    { id: 'stop', msg: '작업을 일시중지할까요?', onOk: () => { statusBuffer.value = status.value; status.value = status_list.find((e) => e.value === 'paused'); dialog.value = null; }, onNo: closeDialog },
    { id: 'harvest', msg: '이 토마토를 수확할까요?', onOk: harvest_tomato, onNo: dont_harvest_tomato },
    { id: 'discard', msg: '이 토마토를 버릴까요?', onOk: discard_tomato, onNo: dont_discard_tomato },
    { id: 'missed_tomato', msg: '놓친 토마토가 있나요?', onOk: finding_missed_tomato, onNo: dont_find_missed_tomato },
]

const isCorrecting = ref(false)

function emergency_stop() {
    if (status.value !== 'stop') {
        statusBuffer.value = status.value
        status.value = status_list.find((e) => e.value === 'paused')
    }
}

function harvest_tomato() {
    isCorrecting.value = true
    dialog.value = 'harvest_true'
    status.value = status_list.find((e) => e.value === 'harvesting')
    setTimeout(() => {
        dialog.value = null
        isCorrecting.value = false
    }, 5000)
}

function dont_harvest_tomato() {
    dialog.value = 'harvest_false'
    status.value = status_list.find((e) => e.value === 'harvesting')
    isCorrecting.value = true
    setTimeout(() => {
        dialog.value = null
        isCorrecting.value = false
    }, 5000)
}

function discard_tomato() {
    dialog.value = 'discard_true'
    status.value = status_list.find((e) => e.value === 'discarding')
    isCorrecting.value = true

    setTimeout(() => {
        dialog.value = null
        status.value = status_list.find((e) => e.value === 'harvesting')
        isCorrecting.value = false

    }, 5000)
}

function dont_discard_tomato() {
    dialog.value = 'discard_false'
    status.value = status_list.find((e) => e.value === 'harvesting')
    isCorrecting.value = true

    setTimeout(() => {
        dialog.value = null
        isCorrecting.value = false

    }, 5000)
}

function finding_missed_tomato() {
    dialog.value = 'find_missed_tomato'
    status.value = status_list.find((e) => e.value === 'finding_missed_tomato')
}

function dont_find_missed_tomato() {
    dialog.value = 'missed_tomato_false'
    closeDialog()
}

const notice_dialog_list = [
    { id: 'harvest_true', msg: '지금부터 토마토 수확을 시작할게요!' },
    { id: 'harvest_false', msg: '네, 이 토마토는 수확하지 않을게요!' },
    { id: 'discard_true', msg: '이 토마토를 버렸어요!' },
    { id: 'discard_false', msg: '이 토마토는 잘 수확했어요!' },
    { id: 'missed_tomato_true', msg: '확인했어요! 이 토마토를 수확하도록 하겠습니다!' },
    { id: 'missed_tomato_false', msg: '네, 잘 수확하도록 하겠습니다!' },
]

const order_dialog_list = [
    { id: 'find_missed_tomato', msg: '화면에서 놓친 토마토가 있는 부분을 눌러주세요!' },
]

const statusBuffer = ref(null)
function clickRobotBtn(btn) {
    if (btn === 'start') {
        if (status.value.value === 'paused') {
            status.value = statusBuffer.value
        } else {
            status.value = status_list.find((e) => e.value === 'harvesting')
        }
    } else if (btn === 'correct') {
        dialog.value = 'select_correction'
        statusBuffer.value = status.value
        status.value = status_list.find((e) => e.value === 'correcting')
    } else if (btn === 'stop') {
        dialog.value = 'stop'
    } else if (btn === 'finish') {
        dialog.value = 'finish'
    }
}

function closeDialog() {
    dialog.value = null;
    status.value = statusBuffer.value;
}

const showQuitRobotDialog = ref(false)
const quitRobotStatus = ref('ask')

function quitRobot() {
    quitRobotStatus.value = 'quitting'
    setTimeout(() => {
        quitRobotStatus.value = 'quitted'
        setTimeout(() => {
            $router.push('/')
        }, 5000)
    }, 5000)
}


async function startStreaming() {
    $q.loading.show()
    const pc = new RTCPeerConnection({
        iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
    });

    pc.addTransceiver("video", { direction: "recvonly" });
    pc.addTransceiver("video", { direction: "recvonly" });

    pc.ontrack = (event) => {
        console.log("Track received:", event);
        
        const newStream = new MediaStream();
        newStream.addTrack(event.track); // 현재 track 추가

        if (!rosCameraMain.value.srcObject) {
            rosCameraMain.value.srcObject = newStream;
        } else {
            rosCameraSub.value.srcObject = newStream;
        }
    };

    rosCameraMain.value.onloadedmetadata = () => {
        console.log("Video metadata loaded");
    };

    pc.oniceconnectionstatechange = () => {
        console.log("ICE Connection State:", pc.iceConnectionState);
    };

    const offer = await pc.createOffer({
        offerToReceiveVideo: true,
    });
    await pc.setLocalDescription(offer);

    const response = await fetch("http://192.168.50.46:8080/offer", {
        method: "POST",
        body: JSON.stringify({ sdp: offer.sdp, type: offer.type }),
        headers: { "Content-Type": "application/json" },
    });

    const answer = await response.json();
    await pc.setRemoteDescription(new RTCSessionDescription(answer));
}

function videoClicked() {
    if (status.value.value === 'finding_missed_tomato') {
        dialog.value = 'missed_tomato_true'
        status.value = status_list.find((e) => e.value === 'harvesting')
        setTimeout(() => {
            dialog.value = null
            status.value = status_list.find((e) => e.value === 'harvesting')
        }, 5000)
    }
}

onMounted(() => {
    startStreaming().then(() => {
        status.value = status_list.find((e) => e.value === 'moving')
        statusBuffer.value = status.value

        setTimeout(() => {
            status.value = status_list.find((e) => e.value === 'pending')
        }, 5000)

        $q.loading.hide()

    });
})




</script>
  