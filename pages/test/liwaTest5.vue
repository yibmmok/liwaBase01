<script setup lang="ts">
	/*********************************************************
	prog name: 測試SingleCol 元件, author: James Lin, date: 2022/12/05
	To Do: 測試 SingleCol 元件
	**********************************************************/
	import { ref, onMounted } from "vue"
	import { useFetch, createFetch } from "@vueuse/core"	
	import liwa1ColList from "../../components/liwa1ColList"
	import liwa2ColList from "../../components/liwa2ColList"
	import { IconPlusLg, IconTrash, IconX, IconAirplane } from '@iconify-prerendered/vue-bi'

	const siteID = ref('')
	const liwaD5 = ref([])
	const liwaD6 = ref([])
	const liwaD8 = ref([])
	const isConfig = ref(false)
	const origin = ref('')
	const gemType = ref('')
	const acctNM = ref('')
	const bShow = ref(false)
	const step = ref('產地設定')
	const stepNames = ['產地設定', '寶石種類設定', '收費項目設定']

	const loadD5 = async () => {
		let url= window.sessionStorage.getItem('liwaAPIsvr') + "/021_haveD5.php?siteID="+siteID.value	
		const dataD5 = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()	
		liwaD5.value = dataD5.data.value.arrSQL
	}

	const loadD6 = async () => {
		let url= window.sessionStorage.getItem('liwaAPIsvr') + "/021_haveGemType.php?siteID="+siteID.value
		const dataD6 = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()	
		liwaD6.value = dataD6.data.value.arrSQL
	}

	const loadD8 = async () => {
		let url= window.sessionStorage.getItem('liwaAPIsvr') + "/021_haveD8.php?siteID="+siteID.value
		const dataD8 = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()	
		liwaD8.value = dataD8.data.value.arrSQL
	}		

	const renewD5 = (arrD5) => {
		liwaD5.value = []
		arrD5.forEach((item) => {
			let objItem = {
				'label': item.itemNM,
				'value': item.mainID
			}
			liwaD5.value.push(objItem)
		})
		console.log('liwaD5 =', liwaD5.value)
	}

	const renewD6 = (arrD6) => {
		liwaD6.value = []
		arrD6.forEach((item) => {
			let objItem = {
				'label': item.itemNM,
				'value': item.mainID
			}
			liwaD6.value.push(objItem)
		})
		console.log('liwaD6 =', liwaD6.value)
	}

	const renewD8 = (arrD8) => {
		liwaD8.value = []
		arrD8.forEach((item) => {
			let objItem = {
				'label': item.item2,
				'value': item.item1
			}
			liwaD8.value.push(objItem)
		})
		console.log('liwaD8 =', liwaD8.value)
	}		

	const toggleConfig = () => {
		isConfig.value = true
		bShow.value = false
	}

	const closeConfig = () => {
		bShow.value = true
		isConfig.value = false
	}

	onMounted(() => {
		siteID.value = window.sessionStorage.getItem('liwaSiteID')
		loadD5()
		loadD6()
		loadD8()
		bShow.value = true
	})

	definePageMeta({
	  title: 'liwa1ColList 測試',
	  layout: "default",
	})		

</script>

<template>
	<div class="w-full h-auto bg-gray-100">
		<div v-if="isConfig==false" class="w-[800px] h-48 mx-auto ">
			<FormKit 
				label="產地"
				type="select"
				v-model="origin"
				:options="liwaD5"
			/>	
			<FormKit 
				label="寶石種類"
				type="select"
				v-model="gemType"
				:options="liwaD6"
			/>	
			<FormKit 
				label="收費項目"
				type="select"
				v-model="acctNM"
				:options="liwaD8"
			/>						
		</div>
		<div class="w-20 h-8 m-2 text-center bg-green-100 rounded-2xl" @click="toggleConfig">設定</div>			
	</div>
	<Teleport to="body">
		<div v-if="isConfig" class="w-full h-auto bg-gray-100 absolute top-[200px] left-0">
			<div class="w-[900px] h-full mx-auto my-0 bg-white relative">
				<ul class="steps">
					<li
				    	v-for="stepName in stepNames"
				    	class="step"
				    	@click="step = stepName"
				    	:data-step-active="step === stepName"
				    >
				    	{{ stepName }}
					</li>
				</ul>
				<div class="w-full bg-slate-100">
					<section v-show="step == stepNames[0]">
						<liwa1ColList
							:fieldCol="['產地']"
							:gridTitle='產地設定表'
							:phpurl="'021_D5edit.php'"
							@setList="renewD5"
						/>						
					</section>
					<section v-show="step == stepNames[1]">
						<liwa1ColList
							:fieldCol="['寶石種類']"
							:gridTitle='寶石種類設定表'
							:phpurl="'021_D6edit.php'"
							@setList="renewD6"
						/>							
					</section>
					<section v-show="step == stepNames[2]">
						<liwa2ColList
							:headCol="['科目編號','收費項目']"
							:gridTitle='收費項目設定表'
							:phpurl="'031_D5edit.php'"
							@setList="renewD8"
						/>							
					</section>					
				</div>
				<div class="w-8 h-8 absolute top-4 right-4"
					@click="closeConfig"
				>
					<IconX class="w-7 h-7 text-red-400 font-bold" />
				</div>
			</div>
		</div>
	</Teleport>
</template>

<style scope>
	@import "../../assets/css/multistep-form.css";
</style>