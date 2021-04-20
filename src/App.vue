<template>
	<div>
		<el-tabs type="border-card">
		  <el-tab-pane>
		    <span slot="label"><i class="el-icon-location"></i> 高德地图Demo</span>
		    <GDMap></GDMap>
		  </el-tab-pane>
		 <!-- <el-tab-pane>
		    <span slot="label"><i class="el-icon-crop"></i> 多边形的绘制和编辑</span>
			<PolygonMap></PolygonMap>
		  </el-tab-pane>
		  <el-tab-pane>
		     <span slot="label"><i class="el-icon-crop"></i> 矩形的绘制和编辑</span>
		  			<RectangleMap></RectangleMap>
		   </el-tab-pane> -->
		   <el-tab-pane>
		      <span slot="label"><i class="el-icon-crop"></i> 鼠标工具绘制矢量图形</span>
		   			<MouseTool></MouseTool>
		    </el-tab-pane>
		</el-tabs>
	</div>
</template>

<script>
	import GDMap from './module/GDMap.vue'
	import PolygonMap from './module/PolygonMap.vue'
	import RectangleMap from './module/RectangleMap.vue'
	import MouseTool from './module/MouseTool.vue'
	export default {
		components:{
		    GDMap,PolygonMap,RectangleMap,MouseTool
		}
	}
</script>

<style>

</style>
