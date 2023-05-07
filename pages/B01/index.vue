<script setup lang="ts">
	/*********************************************************
	prog name: 表格設定列表, author: James Lin, date: 2022/02/06

	**********************************************************/
	import type { Ref } from "vue"
	import { ref, reactive, onMounted, computed } from "vue"
	import queryString from "query-string"
	import { useFetch, createFetch, useStorage, useTitle } from "@vueuse/core"
	import banner from "../../components/banner"
	import { useShowmode } from "../../composables/use-showmode"
	import liwaTblB01 from "../../components/liwaTblB01"
	// import liwaPages from "../../components/liwaPages"
	import { IconPlusLg, IconSearch, IconX } from '@iconify-prerendered/vue-bi'

	const APIsvr = ref('')
	const liwaData = ref({})
	const liwaDetail = ref([])
	const sortIndex = ref(0)
	const progName = ref('表格設定列表')
	const proglink = ref('/B01')
	const detailFlg = ref(false)
	const detailKey = ref('')
	const isFilter = ref(false)
	const filters = ref({})
	const bShowTbl = ref(false)
	const sProgID = ref('')
	const tblParam = ref('')
	const isDetail = ref(false)
	const Dkeydata = ref([])
	const action = ref('view')
	const mainID = ref('')

	const Imgsvr = ref('')
	const paramstr = ref('')


	const router = useRouter()
	// liwaMsg 初始值
	const isMsg = ref(false)
	const objMsg = reactive({
		title: '',
		body: '',
		modalType: 1
	})

	const queryTbl = () => {
		if (!bShowTbl.value) {
			if (sProgID.value) {
				let keydata = {
					"JWT": window.localStorage.getItem('liwaJWT'),
					"progID": sProgID.value
				}
				paramstr.value = JSON.stringify(keydata)
				bShowTbl.value = true	
			}
		} else {
			sProgID.value = ''
			bShowTbl.value = false
		}
	}

	const setDetail = (arrKeys) => {
		action.value = arrKeys[0].action
		mainID.value = arrKeys[0].mainID
		Dkeydata.value = {
			"JWT":window.localStorage.getItem('liwaJWT'),
			"mainID": mainID.value
		}
		if (action.value == 'edit') loadDetail()
		else clearData()
	}

	const loadDetail = async () => {
		let sQuery = queryString.stringify(Dkeydata.value)		
		let url = `${APIsvr.value}/B01_haveDetail.php?${sQuery}`
		const data = await useFetch(url, {method: 'GET'}, {refetch: true}).get().json()
		liwaDetail.value = data.data.value.arrSQL[0]
		liwaDetail.value.isOrder = (Number(liwaDetail.value.isOrder) == 1)? true: false
		liwaDetail.value.canEdit = (Number(liwaDetail.value.canEdit) == 1)? true: false
		isDetail.value = true
	}

	const clearData = () => {
		let tmpData = {
			"mainID":"",
			"progID": sProgID,
			"colNM":"",
			"colField":"",
			"valField":"",
			"fieldPic":"",
			"fieldType":"",
			"isOrder":true,
			"canEdit":true,
			"headCSS":"",
			"bodyCSS":"",
			"bodyOptions":"",
			"slink":"",
			"iIndex":0
		}
		action.value = 'add'
		liwaDetail.value = tmpData
		isDetail.value = true
	}

	const saveData = async (idx) => {
		bShowTbl.value = false
		liwaDetail.value.JWT = window.localStorage.getItem('liwaJWT');
		liwaDetail.value.mainID = mainID.value
		liwaDetail.value.action = action.value
		liwaDetail.value.isOrder = (liwaDetail.value.isOrder)?1: 0
		liwaDetail.value.canEdit = (liwaDetail.value.canEdit)?1: 0
		let datastr = JSON.stringify(liwaDetail.value)
		// console.log('datastr =', datastr)
	    const useMyFetch = createFetch({
	      baseUrl: APIsvr.value,
	      fetchOptions: {
	        mode: 'cors',
	        headers: new Headers({
	          'Content-Type': 'multipart/form-data'
	        }),
	        body: datastr
	      }
	    })
	    const { data } = await useMyFetch('B01_edit.php').post().json()
	    let msg = data.value.message
	    isDetail.value = false
	    if (msg) {
	    	showMsg('存檔錯誤', msg, 1)
	    } 
	    bShowTbl.value = true	    
	}

	const closeBox = () => {
		isDetail.value = false
	}

	// 設定 liwaMsg starts
	const showMsg = (sTitle, sBody, iType = 1) => {
		objMsg.title = sTitle
		objMsg.body = sBody
		objMsg.modalType = iType
		isMsg.value = true
	}

	const hideMsg = () => {
		isMsg.value = false
	}

	const confirmOK = () => {
		isMsg.value = false
	}	
	// 設定 liwaMsg ends

	onMounted(() => {
		useHead({title:'表格設定列表'})
		APIsvr.value = window.sessionStorage.getItem('liwaAPIsvr')
    	Imgsvr.value = window.sessionStorage.getItem('liwaImgsvr')

	})

	definePageMeta({
	  layout: "default",
	  colorMode: "light"
	})	
</script>

<template>
<banner
	:progname="progName"
	:proglink="proglink"
	:detailflg="detailFlg"
	:detailkey="detailKey"
></banner>
<div class="w-full mt-4">
	<div class="w-[1024px] px-8 mx-auto flex flex-row">
		<div class="px-4 py-2">ProgID:</div>
		<div class="w-96 px-4 box-border border-b-2 border-b-slate-300">
			<input type="text" class="searchCol w-full" v-model="sProgID"
				@keyup.enter="queryTbl()"
			/>
		</div>
		<div class="w-24 mx-4 py-2 bg-emerald-800 text-white text-center rounded-xl" @click="queryTbl()">查詢</div>		
	</div>
</div>
<div v-if="bShowTbl" class="w-full min-h-[calc(100vh_-_130px)] relative">
	<liwaTblB01
		:tblTitle="'格式設定列表'"
		:progID="'B01_M'"
		:viewUrl="'B01_edit.php'"
		:dataUrl="'B01_edit.php'"
		:params="paramstr"
		:canEdit="false"
		@setDetail="setDetail"
	/>	
</div>
<Teleport to="body">
	<div v-if="isMsg" 
		class="w-full h-full absolute top-[130px] left-0 bg-slate-100 z-[500]"
	>
		<liwaMsg 
		  	:msgTitle="objMsg.title"
		  	:msgBody="objMsg.body"
		  	:modalType="objMsg.modalType"
		  	@hideMsg="hideMsg"
		  	@confirmOK="confirmOK"
		/> 			
	</div>		
</Teleport>	 
<Teleport to="body">
	<div v-if="isDetail==true"
		 class="w-full h-full absolute top-[130px] left-0 bg-slate-100 z-[500]"
	>
		<div class="w-full lg:w-[1024px] lg:mx-auto relative">
			<div class="absolute top-4 right-2 z-50" @click="closeBox()">
				<IconX class="w-11 h-11 text-red-400 font-bold" />
			</div>
			<div class="w-full p-2 absolute left-0 bg-transparent flex flex-col" >
				<div class="w-full relative">
					<FormKit 
					form-class="px-4 pt-8 bg-violet-100 "
					type="form"
					v-model="liwaDetail"
					:form-class="submitted? 'hidden': 'block'"
					submit-label="存檔"
					@submit="saveData(index)"
					>
						<div class="w-full px-0 py-2 flex flex-col lg:flex-row">
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="colNM"
						          label="標題欄名字*"
						          type="text"
						          placeholder="請設定標題欄名字"
						          help="請設定標題欄名字"
						          validation="required"
						        />
				        	</div>
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="colField"
						          label="資料欄位*"
						          type="text"
						          placeholder="請設定資料欄位"
						          help="請設定資料欄位"
						          validation="required"
						        />
				        	</div>	
						</div>	
						<div class="w-full px-0 py-2 flex flex-col lg:flex-row">
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="valField"
						          label="下拉選單顯示欄位"
						          type="text"
						          placeholder="請設定下拉選單顯示欄位"
						          help="請設定下拉選單顯示欄位"
						        />
				        	</div>
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="fieldPic"
						          label="圖檔路徑欄位"
						          type="text"
						          placeholder="請設定圖檔路徑顯示欄位"
						          help="請設定圖檔路徑顯示欄位"
						        />
				        	</div>	
						</div>	
						<div class="w-full px-0 py-2 flex flex-col lg:flex-row">
				        	<div class="w-full lg:w-1/2 pr-4 flex flex-row">
						        <FormKit
						          name="isOrder"
						          label="是否排序"
						          type="checkbox"
						          :value="true"
						          help="本欄是否允許排序?"
						          wrapper-class="flex flex-row"
						          inner-class="outline-none border-none"
						          label-class="ml-2"
						        />
				        	</div>
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="canEdit"
						          label="可否編輯"
						          type="checkbox"
						          :value="true"
						          help="本欄是否在編輯模式允許編輯?"
						          wrapper-class="flex flex-row"
						          inner-class="outline-none border-none"
						          label-class="ml-2"
						        />
				        	</div>	
						</div>
				        <FormKit
				          name="fieldType"
				          label="欄位型態*"
				          type="text"
				          placeholder="請設定編輯模式的欄位型態"
				          help="請設定編輯模式的欄位型態"
				          validation="required"
				        />	
				        <FormKit
				          name="headCSS"
				          label="標題欄CSS*"
				          type="text"
				          placeholder="標題欄的CSS"
				          help="請設定標題欄的CSS, 要符合Tailwind CSS的語法"
				          validation="required"
				        />
				        <FormKit
				          name="bodyCSS"
				          label="資料欄CSS"
				          type="text"
				          placeholder="資料欄的CSS"
				          help="請設定資料欄的CSS, 要符合Tailwind CSS的語法"
				        />
				        <FormKit
				          name="bodyOptions"
				          label="下拉選項的下拉內容"
				          type="text"
				          placeholder="下拉選項的下拉內容"
				          help="請設定fieldType=liwaDrop的下拉選項內容, 必須為JSON物件的字串值"
				        />					        
						<div class="w-full px-0 py-2 flex flex-col lg:flex-row">
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="slink"
						          label="欄位連結"
						          type="text"
						          placeholder="欄位連結"
						          help="請設定欄位連結, 例如:/041/, 不需加上mainID"
						        />
				        	</div>
				        	<div class="w-full lg:w-1/2 pr-4">
						        <FormKit
						          name="iIndex"
						          label="欄位排列順序*"
						          type="number"
						          placeholder="1"
						          help="請設定欄位排列順序, 從1開始, 不得重覆"
						          validation="required"
						        />
				        	</div>	
						</div>						
					</FormKit>
				</div>
			</div>										
		</div>
	</div>
</Teleport>
</template>

<style scope>
	.searchCol {
		line-height:40px;
	}

	.searchCol:focus {
		outline:none;
	}
</style>