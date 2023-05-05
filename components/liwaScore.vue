<script setup lang="ts">
	/*********************************************************
	prog name: 物件評分元件, author: James Lin, date: 2022/11/26
	Todo: 1. 傳入參數->progressCSS, 
		  2. 
	**********************************************************/	
	import { ref, reactive, onMounted, computed } from "vue"
	import { useFetch, createFetch, useTitle } from "@vueuse/core"
	import queryString from "query-string"
	import { IconX } from '@iconify-prerendered/vue-bi'
	import gemSysData from "../static/gemSys.json"	

	const emits = defineEmits(["setScore"])
	const APIsvr = ref('')
	const error = ref('')
	const liwaSel1 = ref([])
	const liwaDataA = ref([])
	const liwaDataB = ref([])
	const liwaDataC = ref([])
	const liwaDataD = ref([])
	const actvGemSys = ref('A')
	const actvDiamondType = ref('白鑽')
	const actvDiamondColor = ref('天然色')

	const state = reactive({
		'gemSys': '',
		'iTotal': 0,
		'D7':[]
	})

	const getTotalScore = () => {
		state.iTotal = 0
		state.D7 = []
		let liwaWorking = []
		switch (state.gemSys) {
			case 'A': {
					liwaWorking = [...liwaDataA.value]
				}
				break
			case 'B': {
					liwaWorking = [...liwaDataB.value]
				}
				break
			case 'C': {
					liwaWorking = [...liwaDataC.value]
				}
				break
			case 'D': {
					liwaWorking = [...liwaDataD.value]
				}
				break		
		}
		liwaWorking.forEach((item) => {
			let sAns = item.Ans
			let objDetail = {
				'QuesID':item.QuesID,
				'Ques':item.Ques,
				'AnsID':sAns
			}
			state.D7.push(objDetail)
			if (sAns) {
				let arrOpt = [...item.arrOption]
				const res = arrOpt.filter((m) => m.value == sAns)
				state.iTotal = state.iTotal + Number(res[0].iScore)	
			}
		})
	
		if (state.iTotal > 100) state.iTotal = 100		
		emits("setScore", state)
	}	

	const loadOptions = async (sSys) => {
		let keydata = {
			'JWT': window.localStorage.getItem('liwaJWT'),
			'gemSys':sSys
		}
		let sQuery = queryString.stringify(keydata)		
		let url = `${APIsvr.value}/021_haveOptions.php?${sQuery}`
		const data = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()
		switch (sSys) {
			case 'A': liwaDataA.value = data.data.value.arrSQL
				break
			case 'B': liwaDataB.value = data.data.value.arrSQL
				break
			case 'C': liwaDataC.value = data.data.value.arrSQL
				break
			case 'D': liwaDataD.value = data.data.value.arrSQL
				break
		}
	}

	onMounted(() => {
		APIsvr.value = window.sessionStorage.getItem('liwaAPIsvr')
		loadOptions('A')
		loadOptions('B')
		loadOptions('C')
		loadOptions('D')
		state.gemSys = 'A'
		liwaSel1.value = gemSysData
	})		

</script>

<template>
<div v-if="error">{{ error }}</div>
<div class="w-full bg-slate-300 px-4 py-2">
	<div class="barPanel h-16 rounded-3xl ml-4 mb-2 px-1 flex flex-row justify-between relative">
		<div class="w-72 h-16 text-center mx-auto">
			<span class="mt-2 mr-2 float-left ">所屬評分系統: </span>
			<FormKit
				name="gemSys"
				type="select"
				label=""
				outer-class="-pt-1"
				v-model="state.gemSys"
				:options="liwaSel1"
			/>
		</div>
	</div>	
	<div v-if="state.gemSys=='A' && liwaDataA.length > 0" class="w-full mx-auto bg-white p-4">
		<FormKit 
			name="DiamondType"
			type="radio"
			label="鑽石類別"
			v-model="actvDiamondType"
			:options="['白鑽', '彩鑽']"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 顏色 ==</div>	
		<div v-if="actvDiamondType=='白鑽'">
			<FormKit
				:name="liwaDataA[0].QuesID"
				type="radio"
				:label="liwaDataA[0].Ques"
				v-model="liwaDataA[0].Ans"
				:options="liwaDataA[0].arrOption"
			/>
		</div>
		<div v-else>
			<FormKit 
				name="DiamondClorType"
				type="radio"
				label="天然色/處理色"
				v-model="actvDiamondColor"
				:options="['天然色', '處理色']"
			/>				
			<div v-if="actvDiamondColor=='天然色'">
				<FormKit
					:name="liwaDataA[1].QuesID"
					type="radio"
					:label="liwaDataA[1].Ques"
					v-model="liwaDataA[1].Ans"
					:options="liwaDataA[1].arrOption"
				/>
				<FormKit
					:name="liwaDataA[2].QuesID"
					type="radio"
					:label="liwaDataA[2].Ques"
					v-model="liwaDataA[2].Ans"
					:options="liwaDataA[2].arrOption"
				/>					
			</div>
			<div v-else>
				<FormKit
					:name="liwaDataA[3].QuesID"
					type="radio"
					:label="liwaDataA[3].Ques"
					v-model="liwaDataA[3].Ans"
					:options="liwaDataA[3].arrOption"
				/>					
			</div>
		</div>
		<div class="w-full h-8 text-blue-500 font-bold">== 淨度 ==</div>
		<FormKit
			:name="liwaDataA[4].QuesID"
			type="radio"
			:label="liwaDataA[4].Ques"
			v-model="liwaDataA[4].Ans"
			:options="liwaDataA[4].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 車工 ==</div>
		<div v-if="actvDiamondType=='白鑽'">
			<FormKit
				:name="liwaDataA[5].QuesID"
				type="radio"
				:label="liwaDataA[5].Ques"
				v-model="liwaDataA[5].Ans"
				:options="liwaDataA[5].arrOption"
			/>
			<FormKit
				:name="liwaDataA[6].QuesID"
				type="radio"
				:label="liwaDataA[6].Ques"
				v-model="liwaDataA[6].Ans"
				:options="liwaDataA[6].arrOption"
			/>
			<FormKit
				:name="liwaDataA[7].QuesID"
				type="radio"
				:label="liwaDataA[7].Ques"
				v-model="liwaDataA[7].Ans"
				:options="liwaDataA[7].arrOption"
			/>									
		</div>
		<div v-else>
			<FormKit
				:name="liwaDataA[8].QuesID"
				type="radio"
				:label="liwaDataA[8].Ques"
				v-model="liwaDataA[8].Ans"
				:options="liwaDataA[8].arrOption"
			/>
			<FormKit
				:name="liwaDataA[9].QuesID"
				type="radio"
				:label="liwaDataA[9].Ques"
				v-model="liwaDataA[9].Ans"
				:options="liwaDataA[9].arrOption"
			/>						
		</div>
		<div class="w-full h-8 text-blue-500 font-bold">== 稀有度 ==</div>
		<FormKit
			:name="liwaDataA[10].QuesID"
			type="radio"
			:label="liwaDataA[10].Ques"
			v-model="liwaDataA[10].Ans"
			:options="liwaDataA[10].arrOption"
		/>
		<FormKit
			:name="liwaDataA[11].QuesID"
			type="radio"
			:label="liwaDataA[11].Ques"
			v-model="liwaDataA[11].Ans"
			:options="liwaDataA[11].arrOption"
		/>
		<FormKit
			:name="liwaDataA[12].QuesID"
			type="radio"
			:label="liwaDataA[12].Ques"
			v-model="liwaDataA[12].Ans"
			:options="liwaDataA[12].arrOption"
		/>
	</div>
	<div v-if="state.gemSys=='B' && liwaDataB.length > 0" class="w-full mx-auto bg-white p-4">
		<div class="w-full h-8 text-blue-500 font-bold">== 顏色 ==</div>	
		<FormKit
			:name="liwaDataB[0].QuesID"
			type="radio"
			:label="liwaDataB[0].Ques"
			v-model="liwaDataB[0].Ans"
			:options="liwaDataB[0].arrOption"
		/>
		<FormKit
			:name="liwaDataB[1].QuesID"
			type="radio"
			:label="liwaDataB[1].Ques"
			v-model="liwaDataB[1].Ans"
			:options="liwaDataB[1].arrOption"
		/>
		<FormKit
			:name="liwaDataB[2].QuesID"
			type="radio"
			:label="liwaDataB[2].Ques"
			v-model="liwaDataB[2].Ans"
			:options="liwaDataB[2].arrOption"
		/>					
		<div class="w-full h-8 text-blue-500 font-bold">== 淨度 ==</div>
		<FormKit
			:name="liwaDataB[3].QuesID"
			type="radio"
			:label="liwaDataB[3].Ques"
			v-model="liwaDataB[3].Ans"
			:options="liwaDataB[3].arrOption"
		/>					
		<div class="w-full h-8 text-blue-500 font-bold">== 車工 ==</div>
		<FormKit
			:name="liwaDataB[4].QuesID"
			type="radio"
			:label="liwaDataB[4].Ques"
			v-model="liwaDataB[4].Ans"
			:options="liwaDataB[4].arrOption"
		/>
		<FormKit
			:name="liwaDataB[5].QuesID"
			type="radio"
			:label="liwaDataB[5].Ques"
			v-model="liwaDataB[5].Ans"
			:options="liwaDataB[5].arrOption"
		/>
		<FormKit
			:name="liwaDataB[6].QuesID"
			type="radio"
			:label="liwaDataB[6].Ques"
			v-model="liwaDataB[6].Ans"
			:options="liwaDataB[6].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 稀有度 ==</div>
		<FormKit
			:name="liwaDataB[7].QuesID"
			type="radio"
			:label="liwaDataB[7].Ques"
			v-model="liwaDataB[7].Ans"
			:options="liwaDataB[7].arrOption"
		/>									
		<FormKit
			:name="liwaDataB[8].QuesID"
			type="radio"
			:label="liwaDataB[8].Ques"
			v-model="liwaDataB[8].Ans"
			:options="liwaDataB[8].arrOption"
		/>
		<FormKit
			:name="liwaDataB[9].QuesID"
			type="radio"
			:label="liwaDataB[9].Ques"
			v-model="liwaDataB[9].Ans"
			:options="liwaDataB[9].arrOption"
		/>						
		<FormKit
			:name="liwaDataB[10].QuesID"
			type="radio"
			:label="liwaDataB[10].Ques"
			v-model="liwaDataB[10].Ans"
			:options="liwaDataB[10].arrOption"
		/>
		<FormKit
			:name="liwaDataB[11].QuesID"
			type="radio"
			:label="liwaDataB[11].Ques"
			v-model="liwaDataB[11].Ans"
			:options="liwaDataB[11].arrOption"
		/>
	</div>	
	<div v-if="state.gemSys=='C' && liwaDataC.length > 0" class="w-full mx-auto bg-white p-4">
		<div class="w-full h-8 text-blue-500 font-bold">== 顏色 ==</div>	
		<FormKit
			:name="liwaDataC[0].QuesID"
			type="radio"
			:label="liwaDataC[0].Ques"
			v-model="liwaDataC[0].Ans"
			:options="liwaDataC[0].arrOption"
		/>
		<FormKit
			:name="liwaDataC[1].QuesID"
			type="radio"
			:label="liwaDataC[1].Ques"
			v-model="liwaDataC[1].Ans"
			:options="liwaDataC[1].arrOption"
		/>
		<FormKit
			:name="liwaDataC[2].QuesID"
			type="radio"
			:label="liwaDataC[2].Ques"
			v-model="liwaDataC[2].Ans"
			:options="liwaDataC[2].arrOption"
		/>					
		<FormKit
			:name="liwaDataC[3].QuesID"
			type="radio"
			:label="liwaDataC[3].Ques"
			v-model="liwaDataC[3].Ans"
			:options="liwaDataC[3].arrOption"
		/>	
		<div class="w-full h-8 text-blue-500 font-bold">== 種水質地 ==</div>
		<FormKit
			:name="liwaDataC[4].QuesID"
			type="radio"
			:label="liwaDataC[4].Ques"
			v-model="liwaDataC[4].Ans"
			:options="liwaDataC[4].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 工藝 ==</div>
		<FormKit
			:name="liwaDataC[5].QuesID"
			type="radio"
			:label="liwaDataC[5].Ques"
			v-model="liwaDataC[5].Ans"
			:options="liwaDataC[5].arrOption"
		/>
		<FormKit
			:name="liwaDataC[6].QuesID"
			type="radio"
			:label="liwaDataC[6].Ques"
			v-model="liwaDataC[6].Ans"
			:options="liwaDataC[6].arrOption"
		/>
		<FormKit
			:name="liwaDataC[7].QuesID"
			type="radio"
			:label="liwaDataC[7].Ques"
			v-model="liwaDataC[7].Ans"
			:options="liwaDataC[7].arrOption"
		/>									
		<FormKit
			:name="liwaDataC[8].QuesID"
			type="radio"
			:label="liwaDataC[8].Ques"
			v-model="liwaDataC[8].Ans"
			:options="liwaDataC[8].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 稀有度 ==</div>
		<FormKit
			:name="liwaDataC[9].QuesID"
			type="radio"
			:label="liwaDataC[9].Ques"
			v-model="liwaDataC[9].Ans"
			:options="liwaDataC[9].arrOption"
		/>
	</div>	
	<div v-if="state.gemSys=='D' && liwaDataD.length > 0" class="w-full mx-auto bg-white p-4">
		<div class="w-full h-8 text-blue-500 font-bold">== 顏色 ==</div>	
		<FormKit
			:name="liwaDataD[0].QuesID"
			type="radio"
			:label="liwaDataD[0].Ques"
			v-model="liwaDataD[0].Ans"
			:options="liwaDataD[0].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 種質 ==</div>
		<FormKit
			:name="liwaDataD[1].QuesID"
			type="radio"
			:label="liwaDataD[1].Ques"
			v-model="liwaDataD[1].Ans"
			:options="liwaDataD[1].arrOption"
		/>
		<FormKit
			:name="liwaDataD[2].QuesID"
			type="radio"
			:label="liwaDataD[2].Ques"
			v-model="liwaDataD[2].Ans"
			:options="liwaDataD[2].arrOption"
		/>					
		<FormKit
			:name="liwaDataD[3].QuesID"
			type="radio"
			:label="liwaDataD[3].Ques"
			v-model="liwaDataD[3].Ans"
			:options="liwaDataD[3].arrOption"
		/>	
		<div class="w-full h-8 text-blue-500 font-bold">== 工藝 ==</div>
		<FormKit
			:name="liwaDataD[4].QuesID"
			type="radio"
			:label="liwaDataD[4].Ques"
			v-model="liwaDataD[4].Ans"
			:options="liwaDataD[4].arrOption"
		/>	
		<FormKit
			:name="liwaDataD[5].QuesID"
			type="radio"
			:label="liwaDataD[5].Ques"
			v-model="liwaDataD[5].Ans"
			:options="liwaDataD[5].arrOption"
		/>
		<FormKit
			:name="liwaDataD[6].QuesID"
			type="radio"
			:label="liwaDataD[6].Ques"
			v-model="liwaDataD[6].Ans"
			:options="liwaDataD[6].arrOption"
		/>
		<FormKit
			:name="liwaDataD[7].QuesID"
			type="radio"
			:label="liwaDataD[7].Ques"
			v-model="liwaDataD[7].Ans"
			:options="liwaDataD[7].arrOption"
		/>
		<div class="w-full h-8 text-blue-500 font-bold">== 稀有度 ==</div>		
		<FormKit
			:name="liwaDataD[8].QuesID"
			type="radio"
			:label="liwaDataD[8].Ques"
			v-model="liwaDataD[8].Ans"
			:options="liwaDataD[8].arrOption"
		/>

	</div>	
	<div class="w-full h-20 bg-white mx-auto my-2">
		<fieldset>
			<legend>物件評分結果</legend>
			<div class="w-72 h-16 mx-auto my-0 text-blue-600 text-center font-bold">
				<div class="w-20 h-10 bg-purple-100 rounded-lg text-center leading-10 float-left cursor-pointer" @click="getTotalScore()">評分</div>
				<span class="text-2xl">{{ state.iTotal }}</span>
			</div>
		</fieldset>
	</div>		
</div>	
</template>