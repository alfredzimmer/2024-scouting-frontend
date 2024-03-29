<script setup lang="ts">
import SingleNumberCard from '@/components/SingleNumberCard.vue';
import TeamScore from '@/components/TeamScore.vue';
import TeamScorePercent from '@/components/TeamScorePercent.vue'
import CycleTime from '@/components/CycleTime.vue'
import type { Ref } from 'vue';
import { ref } from 'vue';
import type { ScoutingDataUploadForm } from './ScoutingView.vue';
import axios from 'axios';
import { apiBaseUrl } from '@/main';
import { useRoute } from 'vue-router';
import type { PitScoutingForm } from './PitScoutingView.vue';

const teamData: Ref<Array<ScoutingDataUploadForm>> = ref([])

const pitTeamData = ref<PitScoutingForm>()

const id = useRoute().params.id

axios.get(apiBaseUrl + '/api/record/team/' + id)
  .then(response => {
    teamData.value = response.data
    teamData.value.sort((a, b) => {
      if (a.matchNumber.split(' ')[0] != b.matchNumber.split(' ')[0]) {
        if (a.matchNumber.split(' ')[0] == 'Prac') {
          return -1
        } else {
          return 1
        }
      }
      return Number(a.matchNumber.split(' ')[1]) - Number(b.matchNumber.split(' ')[1])
    })
  })
  .then(calculatePercent)
  .then(calculateRating)
  .then(calculateCharts)
  .finally(() => {
    axios.get(apiBaseUrl + '/api/pit/team/' + id)
      .then(response => pitTeamData.value = response.data)
      .catch(e => console.log(e))
  })
  .catch(e => console.log(e))

const autoSpeakerSuccessPercent = ref(0)
const autoAmpSuccessPercent = ref(0)
const teleopSpeakerSuccessPercent = ref(0)
const teleopAmpSuccessPercent = ref(0)

const driverAverageRating = ref(0)
const humanPlayerAverageRating = ref(0)
const strategyAverageRating = ref(0)

const averageOffenseSkill = ref(0)
const averageDefenseSkill = ref(0)
const averageCycleTime = ref(0)


const speakerSuccessXAxis: Ref<Array<string>> = ref([])
const speakerSuccessData: Ref<Array<number>> = ref([])

const speakerSuccessPercentXAxis: Ref<Array<string>> = ref([])
const speakerSuccessPercentData: Ref<Array<number>> = ref([])

const ampSuccessXAxis: Ref<Array<string>> = ref([])
const ampSuccessData: Ref<Array<number>> = ref([])

const ampSuccessPercentXAxis: Ref<Array<string>> = ref([])
const ampSuccessPercentData: Ref<Array<number>> = ref([])

const cycleTimeXAxis: Ref<Array<string>> = ref([])
const cycleTimeData: Ref<Array<number>> = ref([])


function calculateCharts() {
  for (const data of teamData.value) {
    speakerSuccessXAxis.value.push(data.matchNumber)
    speakerSuccessData.value.push(data.teleopSpeakerScored)

    ampSuccessXAxis.value.push(data.matchNumber)
    ampSuccessData.value.push(data.teleopAmpScored)

    cycleTimeXAxis.value.push(data.matchNumber)
    cycleTimeData.value.push(data.cycleTime)

    speakerSuccessPercentXAxis.value.push(data.matchNumber)
    if ((data.teleopSpeakerScored + data.teleopSpeakerMissed) != 0) {
      speakerSuccessPercentData.value.push(Math.round((data.teleopSpeakerScored / (data.teleopSpeakerScored + data.teleopSpeakerMissed)) * 100))
    } else {
      speakerSuccessPercentData.value.push(0)
    }

    ampSuccessPercentXAxis.value.push(data.matchNumber)
    if ((data.teleopAmpScored + data.teleopAmpMissed) != 0) {
      ampSuccessPercentData.value.push(Math.round((data.teleopAmpScored / (data.teleopAmpScored + data.teleopAmpMissed)) * 100))
    } else {
      ampSuccessPercentData.value.push(0)
    }
  }
}

function calculateRating() {
  let totalDriverRating = 0
  let totalHumanPlayerRating = 0
  let totalStrategyRating = 0
  let totalOffenseSkill = 0
  let totalDefenseSkill = 0
  let totalCycleTime = 0
  let numberOfNoCycleTime = 0

  for (const data of teamData.value) {
    totalDriverRating += data.driverRating
    totalHumanPlayerRating += data.humanPlayerRating
    totalStrategyRating += data.strategyRating
    totalOffenseSkill += data.offenseSkill
    totalDefenseSkill += data.defenseSkill
    if (data.cycleTime == 0) {
      numberOfNoCycleTime += 1
    }else {
      totalCycleTime += data.cycleTime
    }
  }

  const ratingNumber = teamData.value.length

  driverAverageRating.value = Number((totalDriverRating / ratingNumber).toFixed(2))
  humanPlayerAverageRating.value = Number((totalHumanPlayerRating / ratingNumber).toFixed(2))
  strategyAverageRating.value = Number((totalStrategyRating / ratingNumber).toFixed(2))
  averageOffenseSkill.value = Number((totalOffenseSkill / ratingNumber).toFixed(2))
  averageDefenseSkill.value = Number((totalDefenseSkill / ratingNumber).toFixed(2))
  averageCycleTime.value = Number((totalCycleTime / (ratingNumber - numberOfNoCycleTime)).toFixed(2))
}

function calculatePercent() {
  let totalAutoAmpShoot = 0
  let totalAutoAmpSuccess = 0

  let totalAutoSpeakerShoot = 0
  let totalAutoSpeakerSuccess = 0

  let totalTeleopSpeakerShoot = 0
  let totalTeleopSpeakerSuccess = 0
  
  let totalTeleopAmpShoot = 0
  let totalTeleopAmpSuccess = 0

  for (const data of teamData.value) {
    totalAutoAmpShoot += data.autoAmpScored + data.autoAmpMissed
    totalAutoAmpSuccess += data.autoAmpScored

    totalAutoSpeakerShoot += data.autoSpeakerScored + data.autoSpeakerMissed
    totalAutoSpeakerSuccess += data.autoSpeakerScored

    totalTeleopSpeakerShoot += data.teleopSpeakerScored + data.teleopSpeakerMissed
    totalTeleopSpeakerSuccess += data.teleopSpeakerScored

    totalTeleopAmpShoot += data.teleopAmpScored + data.teleopAmpMissed
    totalTeleopAmpSuccess += data.teleopAmpScored
  }

  if (
    Number.isNaN(totalAutoAmpShoot) 
    || Number.isNaN(totalAutoSpeakerShoot) 
    || Number.isNaN(totalTeleopSpeakerShoot)
    || Number.isNaN(totalTeleopAmpShoot)
  ) {
    autoAmpSuccessPercent.value = 0
    autoSpeakerSuccessPercent.value = 0
    teleopSpeakerSuccessPercent.value = 0
    return
  }
  autoAmpSuccessPercent.value = Math.round((totalAutoAmpSuccess / totalAutoAmpShoot) * 100)
  autoSpeakerSuccessPercent.value = Math.round((totalAutoSpeakerSuccess / totalAutoSpeakerShoot) * 100)
  teleopSpeakerSuccessPercent.value = Math.round((totalTeleopSpeakerSuccess / totalTeleopSpeakerShoot) * 100)
  teleopAmpSuccessPercent.value = Math.round((totalTeleopAmpSuccess / totalTeleopAmpShoot) * 100)
}

function mapYesOrNo(value: boolean | undefined) {
  if (value === undefined) {
    return '未收集'
  }
  if (value) {
    return '是'
  }
  return '否'
}
</script>

<template>
  <v-container>
    <v-row class="ml-6 mr-6 mt-1">
      <v-col>
        <SingleNumberCard
          :data="$route.params.id.toString()"
          title="队伍号"
        ></SingleNumberCard>
      </v-col>
      <v-col>
        <SingleNumberCard
          :data="autoSpeakerSuccessPercent.toString() + '%'"
          title="自动 Speaker 准确率"
        ></SingleNumberCard>
      </v-col>
      <v-col>
        <SingleNumberCard
          :data="autoAmpSuccessPercent.toString() + '%'"
          title="自动 Amp 准确率"
        ></SingleNumberCard>
      </v-col>
      <v-col>
        <SingleNumberCard
          :data="teleopSpeakerSuccessPercent.toString() + '%'"
          title="手动 Speaker 准确率"
        ></SingleNumberCard>
      </v-col>
      <v-col>
        <SingleNumberCard
          :data="teleopAmpSuccessPercent.toString() + '%'"
          title="手动 Amp 准确率"
        ></SingleNumberCard>
      </v-col>
    </v-row>
    <v-row class="ml-9 mr-6 mt-1">
      <v-col cols="2">
        <v-row class="mt-1">
          <SingleNumberCard
            :data="driverAverageRating.toString() + ' / 10'"
            title="Driver 平均得分"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            :data="humanPlayerAverageRating.toString() + ' / 10'"
            title="HP 平均得分"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            :data="strategyAverageRating.toString() + ' / 10'"
            title="战术平均得分"
          ></SingleNumberCard>
        </v-row>
      </v-col>
      <v-col>
        <TeamScore
          title="Speaker 命中数"
          :x-axis="speakerSuccessXAxis"
          :data="speakerSuccessData"
        ></TeamScore>
      </v-col>
      <v-col>
        <TeamScorePercent
          title="Speaker 命中率"
          :x-axis="speakerSuccessPercentXAxis"
          :data="speakerSuccessPercentData"
        ></TeamScorePercent>
      </v-col>
    </v-row>
    <v-row class="ml-9 mr-6 mt-1">
      <v-col cols="2">
        <v-row class="mt-0">
          <SingleNumberCard
            title="平均 Cycle 时间"
            :data="averageCycleTime.toString() + ' 秒'"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            title="平均攻击能力"
            :data="averageOffenseSkill.toString() + ' / 10'"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            title="平均防守能力"
            :data="averageDefenseSkill.toString() + ' / 10'"
          ></SingleNumberCard>
        </v-row>
      </v-col>
      <v-col>
        <TeamScore
          title="Amp 命中数"
          :x-axis="ampSuccessXAxis"
          :data="ampSuccessData"
        ></TeamScore>
      </v-col>
      <v-col>
        <TeamScorePercent
          title="Amp 命中率"
          :x-axis="ampSuccessPercentXAxis"
          :data="ampSuccessPercentData"
        ></TeamScorePercent>
      </v-col>
    </v-row>
    <v-row class="ml-6 mr-6 mt-1">
      <v-col>
        <CycleTime
          title="Cycle 时间 (秒)"
          :x-axis="cycleTimeXAxis"
          :data="cycleTimeData"
        ></CycleTime>
      </v-col>
    </v-row>
    <v-row class="ml-5 mr-8 mt-1">
      <v-col cols="5">
        <v-sheet class="pa-3" elevation="2" width="100%" height="400px" rounded>
          <v-img :src="pitTeamData?.pictureUrl"></v-img>
        </v-sheet>
      </v-col>
      <v-col>
        <v-row class="mt-1 mr-1">
          <SingleNumberCard
            :data="mapYesOrNo(pitTeamData?.canAmp)"
            title="可以放 Amp"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5 mr-1">
          <SingleNumberCard
            :data="mapYesOrNo(pitTeamData?.canSpeaker)"
            title="可以投 Speaker"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5 mr-1">
          <SingleNumberCard
            :data="mapYesOrNo(pitTeamData?.canTrap)"
            title="可以放 Trap"
          ></SingleNumberCard>
        </v-row>
      </v-col>
      <v-col>
        <v-row class="mt-1">
          <SingleNumberCard
            :data="String(pitTeamData?.chassisType)"
            title="底盘类型"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            :data="String(pitTeamData?.cycleTime)"
            title="最好 Cycle 时间"
          ></SingleNumberCard>
        </v-row>
        <v-row class="mt-5">
          <SingleNumberCard
            :data="String(pitTeamData?.autoType)"
            title="自动类型"
          ></SingleNumberCard>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.chart {
  height: 100vh;
}
</style>
