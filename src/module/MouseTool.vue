<template>
	<div id="MouseTool">
		<div>
			<el-button type="primary" @click="initMap">加载地图</el-button>
			<el-button type="primary" @click="addPlugin">引入工具条插件</el-button>
			<el-button type="primary" @click="destroyMap">销毁地图</el-button>
			<el-button type="primary" @click="setLocation">定位</el-button>
			<el-button type="primary" @click="clearEdit">清除</el-button>
			<el-button type="primary" @click="endEdit">关闭绘制</el-button>
		<!-- 	<el-button type="primary" @click="playBack">轨迹回放</el-button>
			<el-button-group>
			  <el-button type="primary" icon="el-icon-edit" @click="startAnimation">开始动画</el-button>
			  <el-button type="primary" icon="el-icon-share" @click="pauseAnimation">暂停动画</el-button>
			  <el-button type="primary" icon="el-icon-delete" @click="resumeAnimation">继续动画</el-button>
			  <el-button type="primary" icon="el-icon-delete" @click="stopAnimation">停止动画</el-button>
			</el-button-group> -->
			<el-dropdown split-button type="primary" @click="playBack">
			  轨迹回放
			  <el-dropdown-menu slot="dropdown">
			    <el-dropdown-item><el-button type="primary" icon="el-icon-video-play" @click="startAnimation">开始动画</el-button></el-dropdown-item>
			    <el-dropdown-item><el-button type="primary" icon="el-icon-video-pause" @click="pauseAnimation">暂停动画</el-button></el-dropdown-item>
			    <el-dropdown-item><el-button type="primary" icon="el-icon-refresh" @click="resumeAnimation">继续动画</el-button></el-dropdown-item>
			    <el-dropdown-item><el-button type="primary" icon="el-icon-circle-close" @click="stopAnimation">停止动画</el-button></el-dropdown-item>
			  </el-dropdown-menu>
			</el-dropdown>
		</div>
		<div v-if="showRadio">
			<el-radio-group v-model="radio" @change="changeRadio">
				<el-radio v-model="radio" label="marker">画点</el-radio>
				<!-- <el-radio v-model="radio" label="polyline">画折线</el-radio> -->
				<el-radio v-model="radio" label="polygon">画多边形</el-radio>
				<el-radio v-model="radio" label="rectangle">画矩形</el-radio>
				<el-radio v-model="radio" label="circle">画圆</el-radio>
			</el-radio-group>
		</div>
		<div id="myPageTop">
		  <el-input
			id="tipinput"
			placeholder="请输入内容"
			v-model="searchData">
			<i slot="prefix" class="el-input__icon el-icon-search"></i>
		  </el-input>
		</div>
		<div id="mouseToolContainer"></div>
	
	</div>
</template>

<script>
	export default {
		data() {
			return {
				map: null,
				radio:'',
				mouseTool:null,
				overlays:[],
				showRadio:false,
				searchData:'',
				animationMarker:null,
				markers:[],
				lineData:[]
			}
		},
		mounted() {
			// this.initMap()
		},
		methods: {
			// 初始化map
			initMap() {
				const self = this;
				
				if(this.map){
					this.destroyMap();
				}
				
				this.map = new AMap.Map('mouseToolContainer', {
					//center: [108.91083, 34.23936], // 设置地图中心点坐标，如果不设置的话则默认显示当前城市
					zoom: 11, //设置地图显示的缩放级别
					//pitch:75, // 地图俯仰角度，有效范围 0 度- 83 度
					//viewMode:'3D', // 地图模式
					// mapStyle: 'amap://styles/dark', //设置地图的显示样式
					lang: 'zh_cn', //设置地图语言类型
				});
				
			    this.mouseTool = new AMap.MouseTool(this.map); 
				
				this.mouseTool.on('draw',function(e){
					if(e.obj){
						console.log(e.obj)
						console.log(e.obj.Ce.position)
					}
					self.overlays.push(e.obj);
				}) 

				this.initAutoSearch();
				
				this.map.on('complete', function() {
					self.showRadio = true;
					// 地图图块加载完成后触发
					self.$message({
						message: '地图加载完成，当前地图中心点为：' + self.map.getCenter(),
						type: 'success'
					});
				});

			},
			initAutoSearch(){
				var autoOptions = {
					input: "tipinput"
				};
				var auto = new AMap.Autocomplete(autoOptions);
				
				var placeSearch = new AMap.PlaceSearch({
				        map: this.map
				});  //构造地点查询类
				
				AMap.event.addListener(auto, "select", function(e){
					placeSearch.setCity(e.poi.adcode);
					placeSearch.search(e.poi.name);  //关键字查询查询
				});//注册监听，当选中某条记录时会触发
			},
			draw(type){
				const self = this;
				switch(type){
					case 'marker':{
						self.mouseTool.marker({
						  //同Marker的Option设置
						});
						break;
					}
					case 'polyline':{
						self.mouseTool.polyline({
						  strokeColor:'#80d8ff'
						  //同Polyline的Option设置
						});
						break;
					}
					case 'polygon':{
						self.mouseTool.polygon({
						  fillColor:'#00b0ff',
						  strokeColor:'#80d8ff'
						  //同Polygon的Option设置
						});
						break;
					}
					case 'rectangle':{
						self.mouseTool.rectangle({
						  fillColor:'#00b0ff',
						  strokeColor:'#80d8ff'
						  //同Polygon的Option设置
						});
						break;
					}
					case 'circle':{
						self.mouseTool.circle({
						  fillColor:'#00b0ff',
						  strokeColor:'#80d8ff'
						  //同Circle的Option设置
						});
						break;
					}
				}
			},
			changeRadio(value){
				console.log(value);
				this.draw(value);
			},
			// 销毁地图
			destroyMap() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				this.map.destroy();
				this.showRadio = false;
			},
			// 添加工具条插件
			addPlugin() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				const self = this;
				AMap.plugin([
					'AMap.ToolBar',
					'AMap.Scale',
					'AMap.OverView',
					'AMap.MapType',
					'AMap.Geolocation',
				], function() {
					// 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
					self.map.addControl(new AMap.ToolBar());

					// 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
					self.map.addControl(new AMap.Scale());

					// 在图面添加鹰眼控件，在地图右下角显示地图的缩略图
					self.map.addControl(new AMap.OverView({
						isOpen: true
					}));

					// 在图面添加类别切换控件，实现默认图层与卫星图、实施交通图层之间切换的控制
					self.map.addControl(new AMap.MapType());

					// 在图面添加定位控件，用来获取和展示用户主机所在的经纬度位置
					self.map.addControl(new AMap.Geolocation());
				});
			},
			clearEdit(){
				// this.map.remove(this.overlays);
				// this.overlays = [];
				this.map.clearMap();
				
			},
			endEdit(){
				this.mouseTool.close(true)//关闭，并清除覆盖物
				this.radio = null;
			},
			// 创建点标记
			createMarker(x, y, title) {
				var marker = new AMap.Marker({
					position: new AMap.LngLat(x, y), // 经纬度对象，也可以是经纬度构成的一维数组[116.39, 39.9]
					// icon: "https://webapi.amap.com/images/car.png",
					title: title
				});
				return marker;
			},
			// 添加点标记
			addMarker(x, y, title) {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				// 创建点标记
				this.marker = this.createMarker(x, y, title);
				// 设置点标记的动画效果，此处为弹跳效果
				this.marker.setAnimation('AMAP_ANIMATION_BOUNCE');
				// 将创建的点标记添加到已有的地图实例：
				this.map.add(this.marker);
				this.map.setFitView();
			},
			// 浏览器定位
			setLocation() {
				const self = this;
				AMap.plugin('AMap.Geolocation', function() {
					var geolocation = new AMap.Geolocation({
						// 是否使用高精度定位，默认：true
						enableHighAccuracy: true,
						// 设置定位超时时间，默认：无穷大
						timeout: 10000,
						// 定位按钮的停靠位置的偏移量，默认：Pixel(10, 20)
						buttonOffset: new AMap.Pixel(10, 20),
						//  定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
						zoomToAccuracy: true,
						//  定位按钮的排放位置,  RB表示右下
						buttonPosition: 'RB'
					})
			
					geolocation.getCurrentPosition()
					AMap.event.addListener(geolocation, 'complete', onComplete)
					AMap.event.addListener(geolocation, 'error', onError)
			
					function onComplete(data) {
						// data是具体的定位信息
						console.log(JSON.stringify(data));
						self.addMarker(data.position.lng, data.position.lat, data.formattedAddress);
			
						self.map.setFitView();
						// self.$message(JSON.stringify(data));
					}
			
					function onError(data) {
						// 定位出错
						self.$message(JSON.stringify(data));
					}
				})
			},
			playBack(){
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				
				this.lineData = [[116.478935,39.997761],[116.478939,39.997825],[116.478912,39.998549],[116.478912,39.998549],[116.478998,39.998555],[116.478998,39.998555],[116.479282,39.99856],[116.479658,39.998528],[116.480151,39.998453],[116.480784,39.998302],[116.480784,39.998302],[116.481149,39.998184],[116.481573,39.997997],[116.481863,39.997846],[116.482072,39.997718],[116.482362,39.997718],[116.483633,39.998935],[116.48367,39.998968],[116.484648,39.999861]];
				
				this.animationMarker = new AMap.Marker({
					map: this.map,
					position: [116.478935,39.997761],
					title:'起点',
					icon: "https://webapi.amap.com/images/car.png",
					offset: new AMap.Pixel(-26, -13),
					autoRotation: true,
					angle:-90,
				});
				// this.animationMarker.setAnimation('AMAP_ANIMATION_BOUNCE');
				// 绘制轨迹
				var polyline = new AMap.Polyline({
					map: this.map,
					path: this.lineData,
					showDir:true,
					strokeColor: "#28F",  //线颜色
					// strokeOpacity: 1,     //线透明度
					strokeWeight: 6,      //线宽
					// strokeStyle: "solid"  //线样式
				});
				
				var passedPolyline = new AMap.Polyline({
					map: this.map,
					// path: lineArr,
					strokeColor: "#AF5",  //线颜色
					// strokeOpacity: 1,     //线透明度
					strokeWeight: 6,      //线宽
					// strokeStyle: "solid"  //线样式
				});
				
				this.animationMarker.on('moving', function (e) {
					passedPolyline.setPath(e.passedPath);
				});
			
				this.map.setFitView();
				
			},
			startAnimation(){
				this.animationMarker.moveAlong(this.lineData, 200);
			},
			pauseAnimation(){
				this.animationMarker.pauseMove();
			},
			resumeAnimation(){
				this.animationMarker.resumeMove();
			},
			stopAnimation(){
				this.animationMarker.stopMove();
			}
			
		}
	}
</script>

<style>
	#mouseToolContainer {
		width: 1200px;
		height: 750px;
		top: 20px;
	}
</style>
