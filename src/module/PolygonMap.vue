<template>
	<div id="PolygonMap">
		<div>
			<el-button type="primary" @click="initMap">加载地图</el-button>
			<el-button type="primary" @click="addPlugin">引入工具条插件</el-button>
			<el-button type="primary" @click="destroyMap">销毁地图</el-button>
			<el-button type="primary" @click="startEdit">开始绘制</el-button>
			<el-button type="primary" @click="endEdit">结束绘制</el-button>
			<el-button type="primary" @click="pointArea">点是否在区域内</el-button>
		</div>
		<div id="polygonContainer"></div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				map: null,
				markerList: [],
				polygon:null,
				polyEditor:null,
				// 创建多边形
				path : [
					[116.169465,39.932670],
					[116.160260,39.924492], 
					[116.186138,39.879817], 
					[116.150625,39.710019],
					[116.183198,39.709920], 
					[116.226950,39.777616], 
					[116.421078,39.810771],
					[116.442621,39.799892],
					[116.463478,39.790066], 
					[116.588276,39.809551],
					[116.536091,39.808859],
					[116.573856,39.839643], 
					[116.706380,39.916740],
					[116.657285,39.934545],
					[116.600293,39.937770],
					[116.540039,39.937968],
					[116.514805,39.982375],
					[116.499935,40.013710],
					[116.546520,40.030443], 
					[116.687668,40.129961], 
					[116.539697,40.080659],
					[116.503390,40.058474],
					[116.468800,40.052578]
				]
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
				
				this.map = new AMap.Map('polygonContainer', {
					//center: [108.91083, 34.23936], // 设置地图中心点坐标，如果不设置的话则默认显示当前城市
					zoom: 11, //设置地图显示的缩放级别
					//pitch:75, // 地图俯仰角度，有效范围 0 度- 83 度
					//viewMode:'3D', // 地图模式
					//mapStyle: 'amap://styles/dark', //设置地图的显示样式
					lang: 'zh_cn', //设置地图语言类型
				});

				// 同时引入工具条插件，比例尺插件和鹰眼插件
				//this.addPlugin();

				this.polygon = new AMap.Polygon({
			        path: this.path,
			        strokeColor: "#FF33FF", 
			        strokeWeight: 6,
			        strokeOpacity: 0.2,
			        fillOpacity: 0.4,
			        fillColor: '#1791fc',
			        zIndex: 50,
			    })
				
				this.map.add(this.polygon)
				 // 缩放地图到合适的视野级别
				this.map.setFitView([this.polygon])
				
				this.polyEditor = new AMap.PolyEditor(this.map, this.polygon);
				
				this.polyEditor.on('addnode', function(ev) {
					console.log('触发事件：addnode')
				})
			
				this.polyEditor.on('adjust', function(ev) {
					console.log('触发事件：adjust')
					// 触发事件的对象
					var target = ev.target;
					// 触发事件的地理坐标，AMap.LngLat 类型
					var lnglat = ev.lnglat;
					// 触发事件的像素坐标，AMap.Pixel 类型
					var pixel = ev.pixel;
					// 触发事件类型
					var type = ev.type;
					
					console.log(JSON.stringify(ev.lnglat))
				})
			
				this.polyEditor.on('removenode', function(ev) {
					console.log('触发事件：removenode')
				})
			
				this.polyEditor.on('end', function(ev) {
					console.log('触发事件： end')
					// event.target 即为编辑后的多边形对象
				})
				
			},
			// 销毁地图
			destroyMap() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				this.map.destroy();
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
			// 创建点标记
			createMarker(x, y, title) {
				var marker = new AMap.Marker({
					position: new AMap.LngLat(x, y), // 经纬度对象，也可以是经纬度构成的一维数组[116.39, 39.9]
					title: title
				});
				return marker;
			},
			// 添加默认点标记
			addDefaultMarker() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				// 创建点标记
				this.marker = this.createMarker(108.910648, 34.243954, '我的位置');
				// 将创建的点标记添加到已有的地图实例：
				this.map.add(this.marker);

			},
			// 添加点标记
			addMarker(x, y, title) {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}
				// 创建点标记
				this.marker = this.createMarker(x, y, title);
				// 将创建的点标记添加到已有的地图实例：
				this.map.add(this.marker);
			},
			// 移除点标记
			removeMarker() {
				if (!this.map) {
					this.$message.warning('请先创建地图');
					return;
				}

				if (!this.marker) {
					this.$message.warning('请先创建点标记');
					return;
				}
				this.map.remove(this.marker);
				this.map.setFitView();
				this.marker = null;
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
			pointArea(){
				if(!this.map){
					this.$message.warning('请先创建地图');
					return;
				}
				var polygon = new AMap.Polygon({
					map: this.map,
					fillOpacity:0.4,
					path: this.path
				});
				if(this.marker){
					this.map.remove(this.marker);
					this.marker = null;
				}
					
				// 创建点
				this.marker = new AMap.Marker({
					map: this.map,
					draggable:true,
					position: [116.566298, 40.014179]
				});
				
				this.computer();
				
				// 为marker绑定拖拽事件
				const self = this;
				this.marker.on('dragging',function(){
					var point = self.marker.getPosition();
					var isPointInRing = AMap.GeometryUtil.isPointInRing(point,self.path);
					console.log(isPointInRing)
					self.marker.setLabel({
						content:isPointInRing?'内部':'外部',
						offset:new AMap.Pixel(20,0)
					});
				})
				this.map.setFitView();
				
			},
			computer(){
				var point = this.marker.getPosition();
				var isPointInRing = AMap.GeometryUtil.isPointInRing(point,this.path);
				console.log(isPointInRing)
				this.marker.setLabel({
					content:isPointInRing?'内部':'外部',
					offset:new AMap.Pixel(20,0)
				});
			},
			startEdit(){
				this.polyEditor.open();
			},
			endEdit(){
				this.polyEditor.close();
			}
		}
	}
</script>

<style>
	#polygonContainer {
		width: 1200px;
		height: 800px;
		top: 20px;
	}
</style>
