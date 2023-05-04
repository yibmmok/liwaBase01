<script setup lang="ts">
	/*********************************************************
	prog name: 測試FormKit select 元件, author: James Lin, date: 2022/12/05
	To Do: 測試 select的options 如何變更選單值
	**********************************************************/
	import { ref, onMounted } from "vue"
	import liwaScore from "../../components/liwaScore"
	import gemSysData from "../../static/gemSys.json"

	const sGemSys = ref('')
	const arrGemSys = ref([])
	const bShow = ref(false)

	const testProc = () => {
		bShow.value = false
		arrGemSys.value.pop()	
		setTimeout(testTmp, 1000)
		console.log('New arrGemSys =', arrGemSys.value)
	}

	const testTmp = () => {
				
		bShow.value = true		
	}

	const testProc_2 = () => {
		bShow.value = false
		bShow.value = true
	}

	const testProc_3 = () => {
		bShow.value = true
		
	}	

	onMounted(() => {

		arrGemSys.value = gemSysData
		console.log('arrGemSys =', arrGemSys.value)
		bShow.value = true
	})

	definePageMeta({
	  title: 'liwaScore 測試',
	  layout: "default",
	})		

</script>

<template>
	<div class="w-full h-auto bg-gray-100">
		<div class="w-[900px] h-full mx-auto my-0 bg-white relative">
			<div class="w-20 h-8 rounded-2xl bg-yellow-100 text-center absolute top-2 right-[16rem]" @click="testProc()">test1</div>
			<div class="w-20 h-8 rounded-2xl bg-yellow-100 text-center absolute top-2 right-[10rem]" @click="testProc_2()">test2</div>
			<div class="w-20 h-8 rounded-2xl bg-yellow-100 text-center absolute top-2 right-[5rem]" @click="testProc_3()">test3</div>
			<div v-if="bShow" class="w-full h-96 pt-12">
				<FormKit 
					name="gemSys"
					type="select"
					label=""
					:options="arrGemSys"
				/>
			</div>
		</div>
	</div>

</template>