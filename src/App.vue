<template>
	<div>
		<el-tabs type="border-card">
		  <el-tab-pane>
		    <span slot="label"><i class="el-icon-location"></i> 高德地图Demo</span>
		    <GDMap></GDMap>
		  </el-tab-pane>
		 <!-- <el-tab-pane>
		    <span slot="label"><i class="el-icon-chat-round"></i> 即时聊天Demo</span>
		  </el-tab-pane> -->
		</el-tabs>
	</div>
	
</template>

<script>
	import GDMap from './module/GDMap.vue'
	export default {
		components:{
		    GDMap
		}
	}
</script>

<style>

</style>
